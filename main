#pragma config FOSC = XT        // Oscillator Selection bits (XT oscillator)
#pragma config WDTE = OFF       // Watchdog Timer Enable bit (WDT disabled)
#pragma config PWRTE = OFF      // Power-up Timer Enable bit (PWRT disabled)
#pragma config BOREN = OFF      // Brown-out Reset Enable bit (BOR disabled)
#pragma config LVP = OFF        // Low-Voltage (Single-Supply) In-Circuit Serial Programming Enable bit (RB3 is digital I/O, HV on MCLR must be used for programming)
#pragma config CPD = OFF        // Data EEPROM Memory Code Protection bit (Data EEPROM code protection off)
#pragma config WRT = OFF        // Flash Program Memory Write Enable bits (Write protection off; all program memory may be written to by EECON control)
#pragma config CP = OFF         // Flash Program Memory Code Protection bit (Code protection off)

// #pragma config statements should precede project file includes.
// Use project enums instead of #define for ON and OFF.

#include <xc.h>

#define _XTAL_FREQ 4000000

#include<pic.h>
#define data_mode 0x04
#define en_bit 0x08
#define slave_1 0x40
#define slave_2 0x42
//***************** LCD Ctrl Functions ******************//
char lcd_data = 0x01;

char *lcd_1 = "LCD - 1";
char *lcd_2 = "LCD - 2";

//******************* I2C Section *********************//
void i2c_start()
{
	SEN = 1;
	while(SEN);
}

void i2c_wait()
{
	while(SSPIF==0);
	SSPIF = 0;
}

void i2c_rep_start()
{
	RSEN = 1;
	while(RSEN);
}

void i2c_NACK()
{
	ACKDT = 1;
	ACKEN = 1;
	while(ACKEN);
}

void i2c_stop()
{
	i2c_wait();
	PEN = 1;
	while(PEN);
}

unsigned char i2c_rx()
{
	RCEN = 1;
	while(!BF);
	return SSPBUF;
}

unsigned char i2c_send(unsigned char c)
{
	SSPBUF = c;
	while(BF);
	i2c_wait();
}

void delay(unsigned int x)
{
	while(x--);
}

//******************* i2c_data_send ******************//
void cmd_send(char add,char c)
{
	c |= en_bit;
	i2c_rep_start();
	delay(5);
	i2c_send(add);
	i2c_send(c);
	i2c_NACK();
	delay(4);
	i2c_stop();

	delay(10);

	c &= 0xf7;
	i2c_rep_start();
	delay(5);
	i2c_send(add);
	i2c_send(c);
	i2c_NACK();
	delay(4);
	i2c_stop();
	delay(5);
}

void data_send(char add,char c)
{
	
	c |= en_bit | data_mode;
	i2c_rep_start();
	delay(5);
	i2c_send(add);
	i2c_send(c);
	i2c_NACK();
	delay(4);
	i2c_stop();

	delay(10);

	c &= 0xf7;
	i2c_rep_start();
	delay(5);
	i2c_send(add);
	i2c_send(c);
	i2c_NACK();
	delay(4);
	i2c_stop();
	delay(5);
}

void cmd(char add,char y)
{
	lcd_data &= 0x0f;
	lcd_data |= (y&0xf0);
	cmd_send(add,lcd_data);
	delay(10);
	lcd_data &= 0x0f;
	lcd_data |= ((y<<4)&0xf0);
	cmd_send(add,lcd_data);
}

void data(char add,char y)
{
	lcd_data &= 0x0f;
	lcd_data |= (y&0xf0);
	data_send(add,lcd_data);
	delay(10);
	lcd_data &= 0x0f;
	lcd_data |= ((y<<4)&0xf0);
	data_send(add,lcd_data);
}

//*************** Initialization Section ****************//
void i2c_init()
{
	TRISC3 = 1;
	TRISC4 = 1;
	SSPBUF = 0x00;
	SSPSTAT = 0xc0;
	SSPCON = 0x28;
	SSPCON2 = 0x00;
	SSPIF = 0;
	SSPADD = 9;
}

void slave_init(char add)
{
	i2c_rep_start();	// Repeated Start Condition 
	delay(5);
	i2c_send(add);		// Slave Address with Write Bit 0100 000(0)
	i2c_send(0x08);		// LCD EN - ON function
	i2c_NACK();			// Not Acknowledge by Master
	delay(10);
	i2c_stop();			// Terminating the communication
	delay(10);
	i2c_rep_start();	// Again Starting the Communication
	delay(5);
	i2c_send(add);		// Slave Address with Write Bit 0100 000(0)
	i2c_send(0x01);		// LCD EN - OFF function ( This condition to be done for 4 bit Mode ( In normal & i2c mode)
	i2c_NACK();			// Not Acknowledge by Master
	delay(10);
	i2c_stop();			// Terminating the communication

/*	i2c_rep_start();	// Starting the communication
	delay(5);
	i2c_send(0x40);		// Sllave Address with Write Bit 0100 000(0)
	i2c_send(lcd_data); // Initial State | LED P1 - ON Indication for proper communication
	i2c_NACK();			// Not Acknowledge by Master
	delay(10);
	i2c_stop();			// Stop Condition */
}

void main()
{
    TRISB=0XFF;
    PORTB=0;
    char y=0x80;
    int flag=0;
    
    
	i2c_init();		//i2c_Initiation Function
	cmd(slave_1,0x28);		// 4 Bit Mode Config | 2 Line
    slave_init(slave_1);	// Slave Initiation with EN ON & OFF Condition for LCD 4 Bit mode set
	cmd(slave_1,0x0c);		// Display ON | Cursor OFF
	cmd(slave_1,0x06);		// Entry Mode
	cmd(slave_1,0x01);
	
	i2c_init();		//i2c_Initiation Function
	cmd(slave_2,0x28);		// 4 Bit Mode Config | 2 Line
    slave_init(slave_2);	// Slave Initiation with EN ON & OFF Condition for LCD 4 Bit mode set
	cmd(slave_2,0x0c);		// Display ON | Cursor OFF
	cmd(slave_2,0x06);		// Entry Mode
	cmd(slave_2,0x01);
	while(1)
	{
        if(RB0==1){
            __delay_ms(100);
            y++;
            if(y>0x8F){
                if(flag==0){
                    flag=1;
                    cmd(slave_1,0x01);
                }else if(flag==1){
                    flag=0;
                    cmd(slave_2,0x01);
                }
                y=0x80;
            }
        }
        if(RB1==1){
            __delay_ms(100);
            y--;
            if(y<0x80){
                if(flag==0){
                    flag=1;
                }else if(flag==1){
                    flag=0;
                }
                y=0x8F;
            }
        }
        if(flag==0){
            cmd(slave_1,0x01);
            cmd(slave_1,y);		// Position
            data(slave_1,'-');
        }
		if(flag==1){
            cmd(slave_2,0x01);
            cmd(slave_2,y);		// Position
            data(slave_2,'-');
        }
		/*cmd(slave_2,0x83);		// Position
		while(*lcd_2 != '\0')
		{
			data(slave_2,*lcd_2);
			delay(10);
			lcd_2++;
		}	*/
	__delay_ms(1000);
	}
}
