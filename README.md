#include "LPC11xx.h" 
	#include <stdio.h>
	#include <string.h>
	#define UART_BPS 9600
	char GcRcvBuf[20];
	uint8_t  table[9]={0x00,0x01,0x03,0x07,0x0f,0x1f,0x3f,0x7f,0x0ff};
	void Delay(uint32_t ulTime)
	{
		uint32_t i;
		i=0;
		while(ulTime--)
		{
			for (i=0;i<5000;i++);
		}
	}
	void ADC_Init(void)
	{
		LPC_SYSCON->SYSAHBCLKCTRL |=(1<<16);
		LPC_IOCON->PIO1_11 &= ~0xBF;
		LPC_IOCON->PIO1_11 |= 0x01;
		LPC_SYSCON->PDRUNCFG &=~(0x01<<4);
		LPC_SYSCON->SYSAHBCLKCTRL |=(0x01<<13);
		LPC_ADC->CR=(1<<7)|
		((SystemCoreClock/1000000-1)<<8)|
		(0<<16)|
		(0<<17)|
		(0<<24)|
		(0<<27);
	}
	void UART_Init(void)
{
	uint16_t usFdiv;
	LPC_SYSCON->SYSAHBCLKCTRL|=(1<<16);
	LPC_IOCON->PIO1_6&=~0x07;
	LPC_IOCON->PIO1_6|=(1<<0);
	LPC_IOCON->PIO1_7&=~0x07;
	LPC_IOCON->PIO1_7|=(1<<0);
	LPC_SYSCON->SYSAHBCLKCTRL|=(1<<12);
	LPC_SYSCON->UARTCLKDIV=0x01;
	LPC_UART->LCR=0x83;
	usFdiv=(SystemCoreClock/LPC_SYSCON->UARTCLKDIV/16)/UART_BPS;
	LPC_UART->DLM=usFdiv/256;
	LPC_UART->DLL=usFdiv%256;
	LPC_UART->LCR=0x03;
	LPC_UART->FCR=0x07;
}
void GPIOInit(void)
{
	LPC_GPIO2 ->DIR |= 0xFFF;			
	
}
void UART_SendByte(uint8_t ucDat)
{
	LPC_UART->THR=ucDat;
	while((LPC_UART->LSR&0x40)==0);
}
void UART_SendStr(char * pucStr)
{
	while(1)
	{
	if(*pucStr=='\0')break;
	UART_SendByte (*pucStr++);
	}
}	
		
void LedInit(void)
{
	LPC_SYSCON->SYSAHBCLKCTRL |=(1<<6);
	LPC_GPIO2->DIR =0xff;
}

int main(void)
{
	uint32_t i;
	uint32_t ulADCData;
	uint32_t ulADCBuf;
	UART_Init ();
	ADC_Init();
	LedInit();
	while(1)
	{
		ulADCData=0;
		for(i=0;i<10;i++)
		{
			LPC_ADC->CR|=(1<<24);
			while((LPC_ADC->DR[7]&0x80000000)==0);
			LPC_ADC->CR|=(1<<24);
			while((LPC_ADC->DR[7]&0x80000000)==0);
			ulADCBuf=LPC_ADC->DR[7];
			ulADCBuf=(ulADCBuf>>6)&0x3ff;
			ulADCData+=ulADCBuf;
		}
	ulADCData=ulADCData/10;
	ulADCData=(ulADCData*3300)/1024;	
	GPIOInit();
	sprintf(GcRcvBuf,"VINO=%4d mV\r\n",ulADCData);
	UART_SendStr(GcRcvBuf);		
		if((0<ulADCData)&&(412>ulADCData))
		 LPC_GPIO2->DATA=table[7];
if((412<ulADCData)&&(825>ulADCData))
		 LPC_GPIO2->DATA=table[6];
if((852<ulADCData)&&(1264>ulADCData))
		 LPC_GPIO2->DATA=table[5];
if((1264<ulADCData)&&(1676>ulADCData))
		 LPC_GPIO2->DATA=table[4];
if((1676<ulADCData)&&(2088>ulADCData))
		 LPC_GPIO2->DATA=table[3];
if((2088<ulADCData)&&(2500>ulADCData))
		 LPC_GPIO2->DATA=table[2];
if((2500<ulADCData)&&(2912>ulADCData))
		 LPC_GPIO2->DATA=table[1];
if((2912<ulADCData)&&(3324>ulADCData))
		 LPC_GPIO2->DATA=table[0];
	Delay(200);
	}
}
