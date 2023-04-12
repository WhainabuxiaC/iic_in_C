# iic_in_C
How to use iic
## 步骤
1. 启动：SCL保持高电平，SDA由高到低跳变
2. 地址帧：从设备唯一地址序列，主设备发送地址数据，匹配的从设备发送ACK，不匹配的从设备无响应，SDA保持高电平
3. 读/写位：发送从设备地址的最低位为读/写位
4. 数据帧：发送端每发送一个字节数据（8bit），后面接受一个ACK/NACK（接收成功/接受不成功）信号
5. 停止：SCL保持高电平，SDA由低到高跳变

'''
//引脚初始化
void IIC_Init(void)
{
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	
	GPIO_InitStruct.Pin = SCL_GPIO_PIN;
	GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
	GPIO_InitStruct.Pull = GPIO_NOPULL;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
	HAL_GPIO_Init(SCL_GPIO, &GPIO_InitStruct);
	
	GPIO_InitStruct.Pin = SDA_GPIO_PIN;
	GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
	GPIO_InitStruct.Pull = GPIO_NOPULL;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
	HAL_GPIO_Init(SDA_GPIO, &GPIO_InitStruct);    
	
	IIC_SCL_H();  //#define IIC_SCL_H()  HAL_GPIO_WritePin(SCL_GPIO,SCL_GPIO_PIN,GPIO_PIN_SET)
	IIC_SDA_H();  //#define IIC_SCL_L()  HAL_GPIO_WritePin(SCL_GPIO,SCL_GPIO_PIN,GPIO_PIN_RESET)
}
void IIC_SDA_OUT()
{
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	
	GPIO_InitStruct.Pin = SDA_GPIO_PIN;
	GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
	GPIO_InitStruct.Pull = GPIO_NOPULL;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
	HAL_GPIO_Init(SDA_GPIO, &GPIO_InitStruct);    
}

void IIC_SDA_IN()
{
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	
	GPIO_InitStruct.Pin = SDA_GPIO_PIN;
	GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
	GPIO_InitStruct.Pull = GPIO_NOPULL;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
	HAL_GPIO_Init(SDA_GPIO, &GPIO_InitStruct);    
    
}
//IIC起始信号
void IIC_Start()
{
	IIC_SDA_OUT();
	
	IIC_SCL_H();
	IIC_SDA_H();        
	HAL_Delay_Us(5);
	IIC_SDA_L();
	HAL_Delay_Us(6);
	IIC_SCL_L();    
}

//IIC停止信号
void IIC_Stop()
{
	IIC_SDA_OUT();
	IIC_SCL_L();
	IIC_SDA_L();    
	IIC_SCL_H();
	HAL_Delay_Us(5);
	IIC_SDA_H();
	HAL_Delay_Us(5);    
}

//主机应答信号
void IIC_ACK()
{
	IIC_SCL_L();
	IIC_SDA_OUT();
	IIC_SDA_L();
	HAL_Delay_Us(2);
	IIC_SCL_H();
	HAL_Delay_Us(5);
	IIC_SCL_L();
}

//主机非应答
void IIC_NACK()
{
	IIC_SCL_L();
	IIC_SDA_OUT();
	IIC_SDA_H();
	HAL_Delay_Us(2);
	IIC_SCL_H();
	HAL_Delay_Us(5);
	IIC_SCL_L();    
}

//等待从机应答
u8 IIC_Wait_Ack()
{
	u8 tempTime = 0;
	IIC_SDA_IN();
	IIC_SDA_H();
	HAL_Delay_Us(1);
	IIC_SCL_H();
	HAL_Delay_Us(1);
	
	while(READ_SDA())
	{
		tempTime++;
		if(tempTime>250)
		{
			IIC_Stop();
			return 1;   //返回1：应答失败  返回0：应答成功 
		}    
	}
	IIC_SCL_L();
	return 0;
}

//发送一个字节
void IIC_Send_Byte(u8 txd)
{
   u8 i = 0;
    
	IIC_SDA_OUT();
	IIC_SCL_L();    
	for(i=0;i<8;i++)
	{
		if((txd&0x80) > 0)
		{
			IIC_SDA_H();        
		}
		else
		{
			IIC_SDA_L();    
		}        
		txd <<= 1;
		IIC_SCL_H();
		HAL_Delay_Us(2);
		IIC_SCL_L();
		HAL_Delay_Us(2);    
	}    
}

//读取一个字节
u8 IIC_Read_Byte(u8 ack)
{
	u8 i=0;
	u8 receive = 0;
	IIC_SDA_IN();
	for(i=0;i<8;i++)
	{
		IIC_SCL_L();
		HAL_Delay_Us(2);
		IIC_SCL_H();
		receive <<= 1;
		if(READ_SDA())
		{
			receive++;        
		}
		HAL_Delay_Us(1);    
	}    
	if(ack == 0)
	{
		IIC_NACK();    
	}
	else
	{
		IIC_ACK();    
	}
	return receive;    
}
'''
