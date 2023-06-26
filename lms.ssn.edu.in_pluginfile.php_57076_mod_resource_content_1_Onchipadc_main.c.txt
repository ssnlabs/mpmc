/***********************************************************************
*   Onchipadc_main.c									  
************************************************************************/
#include <LPC213X.H>

//////////////////////////////////////////
/*---LCD Signal Declaration---*/ 
//////////////////////////////////////////
#define LCD_RS  1<<24 //Port1.24 
#define LCD_RW  1<<16 //Port0.16
#define LCD_EN  1<<17 //Port0.17
#define LCD_DATA 0xFF<<16 //Port1.16 to 1.23
#define LCD_STS 1<<23 //Port1.23

/////////////////////////////////////////
/*--- ADC Signal Declaration */
/////////////////////////////////////////
#define AD0_1 1<< 24
#define CLK_DIV 1<<8
#define PDN 1<<21
#define SOC 1<<24
#define BURST 1<<16
#define DONE 1<<31

//Function declaration
int main(void);
void lcdini(void);
void lsts(void);
void lcdctl(unsigned char val1);
void lputc(unsigned char lcr);
void lputval(unsigned int lbuf);

unsigned int adc_read( unsigned char channel);
void adc_init(void);

void delay (unsigned int k);
 
//////////////////////////
/*----Main Program------*/
//////////////////////////
int main(void)
{
unsigned int tp1;

  IODIR0 = LCD_RW | LCD_EN; //configure LCD r/w & EN as o/p
  IODIR1 = LCD_RS; //Configure RS o/p
  
  PINSEL0 = 0; //Configure Port0 as General Purpose IO  
  PINSEL1 = 0 | AD0_1; // Enable AD0.1
 
  IOCLR0=LCD_EN; IOCLR1=LCD_RS; IOSET0=LCD_RW; /* DISABLE LCD TEMPORALY*/
  delay(1000); //LCD Power-up Delay
  lcdini();   //Initialise LCD 
  adc_init(); //Initialise on-chip ADC
     
 do
  {  tp1 = adc_read(1); // Channel AD0 0.1
     lputval(tp1); 
     delay(500);
     
     lcdctl(0x01); //  Clear Display
     lsts();
  }while(1);   
  
 return 0;
}
/*----------------------------------------------------------*/
void lcdini()
{  
  lcdctl(0x38); 	 /* Function Set 2 LINE 5 X 8 CHAR*/
  delay(5);   			 /* Waits for 5 Msec. */
  lcdctl(0x38); 	 /* Sends Function Set - AGAIN  */
  delay(5);    			 /* Waits for 5 Msec. */
  lcdctl(0x38);   /* Sends Function Set - AGAIN */
  lsts();      			 /* Wait Till BUSY=0 */
  lcdctl(0x38);   /* Sends Function Set - AGAIN*/
  lsts();      			 /* Wait Till BUSY=0 */
  lcdctl(0x04);   /* Display off */
  lsts();      			 /* Wait Till BUSY=0 */
  lcdctl(0x01);   /* Clear Display */
  lsts();      			 /* Wait Till BUSY=0 */
  lcdctl(0x06);   /* Set Entry mode */
  lsts();      			 /* Wait Till BUSY=0 */
  lcdctl(0x0c);   /* Set Display ON */
 }
/*----------------------------------------------------------*/
/******* Checks the LCD Status for busy***************/
void lsts()
{
unsigned long int tp1;
 
 IOCLR1=LCD_RS; IOSET0=LCD_RW;

do{
  IOSET0=LCD_EN;
  tp1 = IOPIN1 & LCD_STS;
  IOCLR0=LCD_EN;
  }while(tp1);

 IOCLR0=LCD_EN;
 IOCLR0=LCD_RW;
}
/*-----------------------------------------------------------*/
void lcdctl(unsigned char val1)
{ unsigned long int dat;
 
 dat = ((unsigned long int)val1) << 16; 

IODIR1 = LCD_RS | LCD_DATA; //Configure RS & DATA as o/p

 /* WRITE COMMAND TO CONTROL REGISTER*/
 IOCLR1 = LCD_RS;
 IOCLR0 = LCD_RW;
 
 IOCLR1 = LCD_DATA;
 IOSET1 = dat;
 
 IOSET0=LCD_EN;
 IOCLR0=LCD_EN;
 IOSET0=LCD_RW;
 IODIR1 = LCD_RS; //Configure RS as o/p & DATA as i/p
 }
/*-----------------------------------------------------------*/
/******* Displays a Character in the LCD ******/
void lputc(unsigned char lcr)
{ unsigned long int dat;

 lsts();

 dat = ((unsigned long int)lcr)<<16;
 IODIR1 = LCD_RS | LCD_DATA; //Configure RS & DATA as o/p

 IOSET1=LCD_RS; IOCLR0=LCD_RW;
 
 IOCLR1 = LCD_DATA;
 IOSET1 = dat;
 
 IOSET0=LCD_EN;
 IOCLR0=LCD_EN;
 
 IOSET0=LCD_RW;
 IODIR1 = LCD_RS; //Configure RS as o/p & DATA as i/p
 }
/*-----------------------------------------------------------*/
void adc_init()
{
unsigned long int ADC_CH;

	 ADC_CH = 0 | 1 << 1; //Channel AD0.1             
     AD0CR = SOC | PDN | CLK_DIV | ADC_CH | BURST ; 
}
/*-----------------------------------------------------------*/
unsigned int adc_read( unsigned char channel) 
{
	  unsigned int aval;
	  unsigned long int val;
   
     if (channel == 1) val = AD0DR1;
     else if (channel == 2) val = AD0DR2;
     else if (channel == 3) val = AD0DR3;
     
     val = val >> 6;
     val = val & 0x3FF; 
     aval = val;
     return (aval);
}	
/*-----------------------------------------------------------*/
void lputval(unsigned int lbuf)
{	unsigned int val;
	unsigned char tp1;
    val=lbuf;
//Seprate thousands   
   tp1 = (val/1000) + '0';
   lputc(tp1);
 
   val = val%1000;
//Hundreds
   tp1 = (val/100) + '0';   
   lputc(tp1);
   
   val=val%100;
//Tens
   tp1 = (val/10) + '0'; 
   lputc(tp1); 
//One   
   tp1 = (val%10) + '0';   
   lputc(tp1);

 }
/*-----------------------------------------------------------*/	
//Delay Program
//Input - delay value in milli seconds
void delay(unsigned int k)  	
{
	unsigned int i,j;
	for (j=0; j<k; j++)
		for(i = 0; i<=800; i++);
}
/*-----------------------------------------------------------*/

