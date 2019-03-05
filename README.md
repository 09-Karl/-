# -
存储器序号循环排序写入的一种方法

void EERPOM_Log_Init(void)
{

	uint8_t digits_num;//个位数
	uint8_t ten_digits_num;//十位数
	uint16_t hundreds_digits_num;	//百位数
	uint16_t digits_address;	//个位数地址
	uint16_t ten_digits_address;	//十位数地址
	uint16_t hundreds_digits_address;	//百位数地址
	uint16_t i;
	uint16_t j;		
	uint16_t address;	
	uint16_t max_address;			
	uint16_t min_address;				
	uint32_t temple_num;	
	uint32_t max_num;	
	
	digits_num = 0;
	ten_digits_num = 0;
	hundreds_digits_num = 0;
	max_num = 1;
	temple_num = 0;
	address = Sys_EPROM.Log_Start_Address; //Sys_EPROM.Log_Start_Address为开始存储地址
	for(i = Sys_EPROM.Log_Start_Address; i < Sys_EPROM.Log_End_Address; i+=EPROM_LOG_LEN)//遍历所有数据区，数据长度固定为EPROM_LOG_LEN
	{
		temple_num = 0;
		for(j = 0; j < EERPOM_LOG_SERIAL_NUM_LEN; j++)//读取当前序号数据-3byte
			temple_num = temple_num <<8 | AT24CXX_Read_One_Byte(i+j);			
		if((digits_num < temple_num) && (temple_num < 10))//获取个位数据最大值和地址
		{
			digits_num = temple_num;
			digits_address = i;
		}
		
		if((ten_digits_num < temple_num) && (temple_num < 100))//获取十位数数据最大值和地址
		{
			ten_digits_num = temple_num;
			ten_digits_address = i;
		}		
		
		if((hundreds_digits_num < temple_num) && (temple_num < 100))//获取百位数据最大值和地址
		{
			hundreds_digits_num = temple_num;
			hundreds_digits_address = i;
		}		
		
		if((max_num <= temple_num) && (temple_num <= EPROM_MAX_SERIESE_NUM))//获取当前序号最大值和地址，EPROM_MAX_SERIESE_NUM最大序号值
		{
			max_num = temple_num;
			address = i;
		}
	}
	
	if((max_num == 1) && (i == Sys_EPROM.Log_End_Address))//扇区为空，首次使用
	{
		Sys_EPROM.Serial_Num = 1;
		Sys_EPROM.Write_Log_Address = Sys_EPROM.Log_Start_Address;
		return;					
	}
	
  //Sys_EPROM.Write_Log_Address为需要写入数据的地址
  //Sys_EPROM.Serial_Num为当前序号值
  
	if(max_num < EPROM_MAX_SERIESE_NUM)
		Sys_EPROM.Serial_Num = max_num + 1;
	else 	if(max_num >= EPROM_MAX_SERIESE_NUM)//存在最大值，循环一次序号，需要重新排序
	{
		if((digits_num <= 9) && (ten_digits_num == 0))//使用个位数最大序号
		{
			Sys_EPROM.Serial_Num = digits_num+1;
			address = digits_address;
		}
		else if((ten_digits_num <= 99) && (hundreds_digits_num == 0))//使用十位数最大序号
		{
			Sys_EPROM.Serial_Num = ten_digits_num+1;
			address = ten_digits_address;			
		}	
		else//使用百位数最大序号
		{
			Sys_EPROM.Serial_Num = hundreds_digits_num+1;
			address = hundreds_digits_address;						
		}				
	}

	address += EPROM_LOG_LEN;//地址向下一个地址偏移
	if(address > Sys_EPROM.Log_End_Address)//偏移地址溢出，返回开始地址
		Sys_EPROM.Write_Log_Address = Sys_EPROM.Log_Start_Address;		
	else
		Sys_EPROM.Write_Log_Address = address;
  ｝
