# LIBFUSION-_C
/*p0.2,P0.3,cclk=96Mhz,pclk-24Mhz,DLL=146,DLM=0,mul=15,dis=1 */
#include<lpc17xx.h>
#define rs (1<<10)
#define en (1<<11)
#define dr (0xFF<<15)
void delay(unsigned int a);
void tx(unsigned char c);
void lcd_init();
void cmd(unsigned char c);
void dat(unsigned char d);
void lcd_str(unsigned char str[20]);
void lcd_num(unsigned int num);
void pll_96mhz();
int main()
{
char val;
lcd_init();
pll_96mhz();
//UART configuration
LPC_SC->PCONP=(1<<3); //enable power
LPC_PINCON->PINSEL0|=(1<<4)|(1<<6); //config P0.2,P0.3 as Tx and Rx
LPC_UART0->LCR=(1<<0)|(1<<1)|(1<<7); //8bit mode. No parity Enable Division
LPC_UART0->DLL=146;
LPC_UART0->DLM=0;
LPC_UART0->FDR=(15<<4)|(1<<0);
LPC_UART0->LCR&=~(1<<7);
while(1)
{
while(!(LPC_UART0->LSR&(1<<0)));
val=LPC_UART0->RBR;
dat(val);
delay(100);
while(!(LPC_UART0->LSR&(1<<5))); //CHECK THR EMPTY OR NOT USING POLLING
LPC_UART0->THR='A';
delay(100);
}
}
void lcd_init()
{
LPC_GPIO0->FIODIR=rs|en|dr;
cmd(0x38);//8bit 2lines
cmd(0x0E);// cusor on,display on
cmd(0x01);//clear display
cmd(0x80); //first row first coloumn
cmd(0xC0); //second row first coloumn
}
void cmd(unsigned char c)
{
 LPC_GPIO0->FIOCLR=dr;
 LPC_GPIO0->FIOSET=(c<<15);
 LPC_GPIO0->FIOCLR=rs; //rs=0;for setting command mode
 LPC_GPIO0->FIOSET=en;
 delay(10);
 LPC_GPIO0->FIOCLR=en;
 delay(10);
 }
 void dat(unsigned char d)
 {
 LPC_GPIO0->FIOCLR=dr;
 LPC_GPIO0->FIOSET=(d<<15);
 LPC_GPIO0->FIOSET=rs; //rs=1;for setting datamode
 LPC_GPIO0->FIOSET=en;
 delay(10);
 LPC_GPIO0->FIOCLR=en;
 delay(10);
 }

 void lcd_str(unsigned char str[20])
 {
 int i;
 for(i=0;str[i]!='\0';i++)
 {
 dat(str[i]);
 }
 }

void lcd_num(unsigned int num)
{
if(num)
{
lcd_num(num/10);
dat(num%10+0x30);
}
}
void pll_96mhz(void){
//enable main oscillator
LPC_SC->SCS=(1<<4)|(1<<5);
//disable pll
LPC_SC->PLL0CON =0;
//give feed signal
LPC_SC->PLL0FEED=0xAA;
LPC_SC->PLL0FEED=0X55;
//check main oscillator stable or not
while(!(LPC_SC->SCS&(1<<6)));
//select clk source from PLL
LPC_SC->CLKSRCSEL=1;
//config PLL with
LPC_SC->PLL0CFG = 0x00F;//(15<<0)|(0<<16);
//enable PLL
  LPC_SC->PLL0CON=1;
LPC_SC->PLL0FEED=0XAA;
LPC_SC->PLL0FEED=0X55;

LPC_SC->CCLKCFG=3;
//check pll locked or not
while(!(LPC_SC->PLL0STAT&(1<<26)));
//enable PLL & connect to cpu
LPC_SC->PLL0CON=(1<<0)|(1<<1);
//feed signal
LPC_SC->PLL0FEED=0XAA;
LPC_SC->PLL0FEED=0X55;
}

void delay(unsigned int a)
{
int i,j;
for(i=0;i<a;i++)
{
for(j=0;j<a;j++);
}
}
