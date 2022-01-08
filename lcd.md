
#include "main.h"

PORTA=0B00000000001;
ADC_HandleTypeDef hadc1;

void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_ADC1_Init(void);
void LCD_init ();
void LCD_ON ();
void LCD_CLEAR ();
void LCD_WRITE (char A[]);


int main(void)
{
  char c[]={};
  float adc;
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_ADC1_Init();
  LCD_init ();
  LCD_ON ();
  while (1)
  {
	HAL_ADC_Start(&hadc1);
	HAL_ADC_PollForConversion(&hadc1,1);
    adc=HAL_ADC_GetValue(&hadc1);
    gcvt(adc,7, c);
    LCD_CLEAR();
    //LCD_WRITE("VALOR ADC 12 BITS");
    LCD_WRITE(c);
    HAL_Delay(500);

  }

}
void LCD_init (){
int i;
//Se realiza el cambio de 8 bits a 4
for(i=0;i<=2;i++){
	GPIOB->ODR=0b00110000;
	HAL_Delay(5);
	GPIOB->ODR=0b00110010;
	HAL_Delay(5);
	GPIOB->ODR=0b00110000;
}
//Se indica el modo de trabajo de 4 bits
GPIOB->ODR=0b00100000;//palabra
HAL_Delay(35);
GPIOB->ODR=0b00100010; //pulso en alto
HAL_Delay(5);
GPIOB->ODR=0b00100000;//pulso en bajo
HAL_Delay(5);

GPIOB->ODR=0b00100010;
HAL_Delay(5);
GPIOB->ODR=0b00100000;
HAL_Delay(5);
GPIOB->ODR=0b11000000;
HAL_Delay(5);
GPIOB->ODR=0b11000010;
HAL_Delay(5);
GPIOB->ODR=0b11000000;
}

void LCD_ON (){
	GPIOB->ODR=0b00000000;
	HAL_Delay(35);
	GPIOB->ODR=0b00000010;
	HAL_Delay(5);
	GPIOB->ODR=0b00000000;
	HAL_Delay(5);
	GPIOB->ODR=0b11110000;
	HAL_Delay(35);
	GPIOB->ODR=0b11110010;
	HAL_Delay(5);
	GPIOB->ODR=0b11110000;
}

void LCD_CLEAR (){
	GPIOB->ODR=0b00000000;
    HAL_Delay(0.010);
	GPIOB->ODR=0b00000010;
	HAL_Delay(0.010);
	GPIOB->ODR=0b00000000;
	HAL_Delay(0.010);
	GPIOB->ODR=0b00010000;
	HAL_Delay(0.010);
	GPIOB->ODR=0b00010010;
	HAL_Delay(0.010);
	GPIOB->ODR=0b00010000;
}

void LCD_WRITE (char A[]){

    int PA,PB,j,k=1,w=0;
    while(k!=0){
    k=A[w];
    w=w+1;
    }
    for(j=0;j<=w-2;j++){
		      PA=A[j]>>4<<4;//parte alta
			  PB=A[j]<<4; //parte baja

			  GPIOB->ODR=PA + 0b1000;
			  HAL_Delay(0.010);
			  GPIOB->ODR=PA + 0b1010;
			  HAL_Delay(0.010);
			  GPIOB->ODR=PA + 0b1000;
			  HAL_Delay(0.010);
			  GPIOB->ODR=PB + 0b1000;
			  HAL_Delay(0.010);
			  GPIOB->ODR=PB + 0b1010;
			  HAL_Delay(0.010);
			  GPIOB->ODR=PB + 0b1000;
	}
}





void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};
  RCC_PeriphCLKInitTypeDef PeriphClkInit = {0};

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI_DIV2;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL2;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }
  /** Initializes the CPU, AHB and APB buses clocks
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_0) != HAL_OK)
  {
    Error_Handler();
  }
  PeriphClkInit.PeriphClockSelection = RCC_PERIPHCLK_ADC;
  PeriphClkInit.AdcClockSelection = RCC_ADCPCLK2_DIV6;
  if (HAL_RCCEx_PeriphCLKConfig(&PeriphClkInit) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
  * @brief ADC1 Initialization Function
  * @param None
  * @retval None
  */
static void MX_ADC1_Init(void)
{



  ADC_ChannelConfTypeDef sConfig = {0};


  hadc1.Instance = ADC1;
  hadc1.Init.ScanConvMode = ADC_SCAN_DISABLE;
  hadc1.Init.ContinuousConvMode = DISABLE;
  hadc1.Init.DiscontinuousConvMode = DISABLE;
  hadc1.Init.ExternalTrigConv = ADC_SOFTWARE_START;
  hadc1.Init.DataAlign = ADC_DATAALIGN_RIGHT;
  hadc1.Init.NbrOfConversion = 1;
  if (HAL_ADC_Init(&hadc1) != HAL_OK)
  {
    Error_Handler();
  }
  /** Configure Regular Channel
  */
  sConfig.Channel = ADC_CHANNEL_7;
  sConfig.Rank = ADC_REGULAR_RANK_1;
  sConfig.SamplingTime = ADC_SAMPLETIME_1CYCLE_5;
  if (HAL_ADC_ConfigChannel(&hadc1, &sConfig) != HAL_OK)
  {
    Error_Handler();
  }

}

/**
  * @brief GPIO Initialization Function
  * @param None
  * @retval None
  */
static void MX_GPIO_Init(void)
{
	GPIO_InitTypeDef GPIO_InitStruct = {0};
  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOA_CLK_ENABLE();

  __HAL_RCC_GPIOB_CLK_ENABLE();
   HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3|GPIO_PIN_4|GPIO_PIN_5|GPIO_PIN_6|GPIO_PIN_7, GPIO_PIN_RESET);

   GPIO_InitStruct.Pin = GPIO_PIN_0|GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3|GPIO_PIN_4|GPIO_PIN_5|GPIO_PIN_6|GPIO_PIN_7;
   GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
   GPIO_InitStruct.Pull = GPIO_NOPULL;
   GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
   HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);


}

/* USER CODE BEGIN 4 */

/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */
  __disable_irq();
  while (1)
  {
  }
  /* USER CODE END Error_Handler_Debug */
}

#ifdef  USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{
  /* USER CODE BEGIN 6 */
  /* User can add his own implementation to report the file name and line number,
     ex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */
