// File: Blinken.c - Blinken einer DIODE
//
// XE167.h muss eingebunden werden, weil dann die symbolischen
// Namen wie z.B. P10_OUT_P0 verwendet werden können und nicht
// Adresse von P10_OUT_P0 verwendet werden muss!
//
#include <XE167F.h>
#define FREQ_MULTIPLIER 1
#define TIMER_FREQ 20 //in KHz*FREQ_MULTIPLIER
volatile unsigned int counter1,counter2;
// Funktion wait() braucht ~1 sek.

typedef unsigned int uword;

void MAIN_vUnlockProtecReg(void)
{
    uword uwPASSWORD;
    SCU_SLC = 0xAAAA; // command 0
    SCU_SLC = 0x5554; // command 1
    uwPASSWORD = SCU_SLS & 0x00FF;
    uwPASSWORD = (~uwPASSWORD) & 0x00FF;
    SCU_SLC = 0x9600 | uwPASSWORD; // command 2
    SCU_SLC = 0x0000; // command 3
}


void MAIN_vLockProtecReg(void)
{
    uword uwPASSWORD;
    SCU_SLC = 0xAAAA; // command 0
    SCU_SLC = 0x5554; // command 1
    uwPASSWORD = SCU_SLS & 0x00FF;
    uwPASSWORD = (~uwPASSWORD) & 0x00FF;
    SCU_SLC = 0x9600 | uwPASSWORD; // command 2
    SCU_SLC = 0x1800; // command 3; new PASSWOR is 0x00
    uwPASSWORD = SCU_SLS & 0x00FF;
    uwPASSWORD = (~uwPASSWORD) & 0x00FF;
    SCU_SLC = 0x8E00 | uwPASSWORD; // command 4
}

void toggle_GP_LED(int LED)
{
    switch (LED) {
        case 0:
            P10_OUT_P0 = ~P10_OUT_P0;
        break;
        case 1:
            P10_OUT_P1 = ~P10_OUT_P1;
        break;
        case 2:
            P10_OUT_P2 = ~P10_OUT_P2;
        break;
        case 3:
            P10_OUT_P3 = ~P10_OUT_P3;
        break;
        case 4:
            P10_OUT_P4 = ~P10_OUT_P4;
        break;
        case 5:
            P10_OUT_P5 = ~P10_OUT_P5;
        break;
        case 6:
            P10_OUT_P6 = ~P10_OUT_P6;
        break;
        case 7:
            P10_OUT_P7 = ~P10_OUT_P7;
        break;
        default:
            break;
    }
}

void set_GP_LEDs(int val)
{
		val = ~val;
		P10_OUT_P0 = val&1;
		P10_OUT_P1 = (val&2)>>1;
		P10_OUT_P2 = (val&4)>>2;
		P10_OUT_P3 = (val&8)>>3;
		P10_OUT_P4 = (val&0x10)>>4;
		P10_OUT_P5 = (val&0x20)>>5;
		P10_OUT_P6 = (val&0x40)>>6;
		P10_OUT_P7 = (val&0x80)>>7;
}

void wait()
{
    counter1 = 0x18; //war vorher 0x40
    while (counter1 > 0)
    {
        counter1--;
        counter2 = 0xFFFF;
        while (counter2 > 0)
        {
            counter2--;
        }
    }
}

 /*
    Bit 15: 1 // Reload from register CAPREL Enabled
    Bit 14: // 0 -> Timer T6 is not cleared on a capture event
    Bit 13: // does nothing
    Bit 12: // [12:11] Selects the basic clock, 
    bit 11: // 00 for f/4
    Bit 10: // Timer T6 Overflow Toggle Latch: Toggles on each overflow/underflow of T6
    bit 9: // Overflow/Underflow Output Enable
    Bit 8: // Timer T6 External Up/Down Enable: 0 -> Input T6EUD is disconnected
    Bit 7: 1 // 1: counts down
    Bit 6: // Timer T6 Run Bit:  0 -> Timer T6 stops   1 -> Timer T6 runs
    Bit 5:0 // Timer T6 Mode Control [5:3]  000 Timer Mode
    Bit 4:0
    Bit 3:0
    Bit 2:
    Bit 1:
    Bit 0:
	  */



/*

void StartTimer()
{
	GPT12E_T6CON|=1<<6;
}

void SetTimer(int time) //in ms
{
	int steps = time*TIMER_FREQ/FREQ_MULTIPLIER;
	GPT12E_CAPREL=steps;
}

void StopTimer()
{
	  GPT12E_T6CON^=1<<6;
}//*/

int a_interrupt=0; // zählt wie oft interrupt ausgelöst wird
void GPT2_viT6() interrupt 0x24 // Problem: wird zu schnell/oft aufgerufen
{
	toggle_GP_LED(1);
	a_interrupt=a_interrupt+1;
}


int switchcount = 0;
void Schalterinterrupt() interrupt 0x10
{
	CC2_T78CON_T8R = 1;
	//switchcount++;
	//set_GP_LEDs(switchcount);
}

void SchalterStop() interrupt 0x12
{
	CC2_T78CON_T8R = 0;
	set_GP_LEDs(CC2_T8>>8);
}

/*
void display_number_LEDs(int num)
{

}
*/

void Prepare_Outputs()
{
	// Pin 0 von Port 10 wird auf "OUT" gesetzt
    P10_IOCR00 = 0x0080;
    P10_IOCR01 = 0x0080;
    P10_IOCR02 = 0x0080;
    P10_IOCR03 = 0x0080;
    P10_IOCR04 = 0x0080;
    P10_IOCR05 = 0x0080;
    P10_IOCR06 = 0x0080;
    P10_IOCR07 = 0x0080;
    // Pin 0 von Port 10 erhält den Wert 1
    P10_OUT = 0xFF;
}

void Prepare_Inputs()
{
	//P2.3 als Input (ohne Pull Up/Down-Widerstand o.ä.)
		P2_IOCR03 = 0x0; 
		P4_IN_P4 = 0x0000;
		CC2_M4|=0x101; // steigende Flanke
		CC2_CC16IC|=0x4C; // 0100 1100
		CC2_CC18IC|=0x4C;
	//Set Bits: 0000 0111 0000 0000
	//Required:  *** ****          
		CC2_T78CON |= 0x0700;
		CC2_T78CON &= 0x87ff;
}

void main(void)
{
	//Allgemeines Setup
    MAIN_vUnlockProtecReg();
		CC2_KSCCFG = 3; 
    MAIN_vLockProtecReg();
    
		PSW_IEN = 1;	
    Prepare_Outputs();
		Prepare_Inputs();
    
    
		
    while (1)
    {
			wait();
    }
}

//vol 2, seite 72  Frequenzen  (14.2.6 GPT2 Clock Signal Control)
