# STM32闹钟的一个进阶使用

##### By Sky.J 			2018.06.23



### 概述：

​	在使用STM32的过程中，我们在项目中可能经常会用到它的闹钟功能，但是对于刚开始接触STM32闹钟时，我就是直接设置一个闹钟，然后等待中断，有时候如果有两个闹钟，我们可以用ALARM A和B,但是有4个5个或者更多的闹钟设置，这时就不知道怎么办了。我就根据我的使用需求想了一个办法（只涉及几点几分，不考虑年月日及周几），如果你也有这样的需求，可以直接使用，如果不是，也希望可以给你留下一个思考的方向。



### 思路：

​	因为我的需求是每天的几点几分有一个闹钟，然后去处理，所以我的思路也很简单，就是将所有闹钟都注册到一个数组里面，然后换算为分钟从小到大进行排序，再根据当前时间去选择我下一个要设置的闹钟是哪一个，比如我已经注册了3个闹钟，时间分别是：200 300 500，当前时间是360，那么我现在选择设置的闹钟肯定是500。所以每次我们当闹钟响了以后，再去用当前时间和数组里面的闹钟时间对比，我们就可以得到下个闹钟需要设定的时间，是不是解决了闹钟设定的问题了呢。当然，有人会说闹钟设定了，我不知道这个闹钟响的时候我需要处理啥事件，很简单，我们在注册闹钟时不仅只注册时间，同时还需要注册闹钟事件标志。比如下方结构体：

```c
typedef enum {
  PRINT_PEACE = 0,
  PRINT_LOVE,
  PRINT_HAVE_FUN
}alarm_process_status_t;

typedef struct {
  uint32_t time;                             //闹钟时间
  alarm_process_status_t process_status;	//闹钟事件
}alarm_info_t;
```



### Linux下的模拟代码

​	这里我写了一个测试代码,为了方便调试和个人习惯，我自己是在linux下写了一个C代码，闹钟响不响，无非是中断是不是产生，至于怎么设置闹钟，这些我觉得你如果需要设置多个闹钟时，你一定已经基本知道闹钟的设置，我这里就不用STM32的实例代码了，我这边都是模拟，但效果是一样的，理解了思路就可以拿去增删改后用了。

```c
#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>
#include <time.h>

#define ALARM_SIZE        10

typedef enum {
  PRINT_PEACE = 0,
  PRINT_LOVE,
  PRINT_HAVE_FUN
}alarm_process_status_t;

typedef enum {
  ALARM_NOSET = 0,
  ALARM_SET
}alarm_set_status_t;

typedef struct {
  uint32_t time;
  alarm_process_status_t process_status;
}alarm_info_t;

typedef struct {
  uint8_t size;					//闹钟注册个数
  alarm_set_status_t set_status;	//闹钟是否设置标志
  alarm_process_status_t process_status;	//当前设置的闹钟响后需要做的事件标志
  alarm_info_t info[ALARM_SIZE];	//存储注册的闹钟
}alarm_ctrl_t;

alarm_ctrl_t alarmctrl;

/*获取当前时间分钟*/
uint32_t get_local_time()
{
  time_t now;
  struct tm *tm_now;
  time(&now);
  tm_now = localtime(&now);
  uint32_t times = tm_now->tm_hour * 60 + tm_now->tm_min;
  return times;
}

/*初始化*/
void alarm_ctrl_init(alarm_ctrl_t *alarm_ctrl)
{
  alarm_ctrl->size = 0;
  alarm_ctrl->set_status = ALARM_NOSET;
}

/*获取闹钟响铃状态 1响 0 不响*/
uint8_t get_alarm_clock()
{
  //对于在STM32中，我们可以在处理中断的文件中定义一个全局变量，当产生中断事变量变为1，否则为0
  //此处测试，故一直返回1
  return 1;
}

/*数组数据展示*/
void display_alarm_info()
{
  alarm_ctrl_t *alarm_ctrl = &alarmctrl;
  uint8_t i;
  printf("show alarm info: \r\n");
  for (i = 0; i < alarm_ctrl->size; i++) {
    printf(" %d ", alarm_ctrl->info[i].time);
  }
  printf("\r\n");
}

/*数组冒泡排序*/
void sort_alarm_info(alarm_info_t *info, uint8_t size)
{
  uint8_t i, j;
  for (i =0; i < size - 1; i++)
  {
    for (j = 0; j < size-1-i; j++) {
      if (info[j].time > info[j+1].time) {
        alarm_info_t temp = info[j];
        info[j] = info[j+1];
        info[j+1] = temp;
      }
    }
  }
}

/*数组插入*/
void insert_alarm_info(alarm_ctrl_t *alarm_ctrl, alarm_info_t *alarm_info)
{
  uint8_t i;
  uint8_t size = alarm_ctrl->size;
  if (size >= ALARM_SIZE) {
    return;
  }
  //相同闹钟禁止插入
  for (i = 0; i < size; i++) {
    if (alarm_ctrl->info[i].time == alarm_info->time) {
      printf("alarm time is exist, error\r\n");
      return ;
    }
  }
  //插入数组数据并通过时间从小到大排序
  alarm_ctrl->info[size].process_status = alarm_info->process_status;
  alarm_ctrl->info[size].time = alarm_info->time;
  alarm_ctrl->size ++;
  sort_alarm_info(alarm_ctrl->info, alarm_ctrl->size);
  display_alarm_info();
}

/*数组删除*/
void delete_alarm_info(alarm_ctrl_t *alarm_ctrl, uint32_t time)
{
  uint8_t i, j, point, flag = 0;
  uint8_t size = alarm_ctrl->size;
  //查找删除的时间点是否存在
  for (i = 0; i < size; i++) {
    if (alarm_ctrl->info[i].time == time) {
      flag = 1;
      point = i;
      break;
    }
  }
  if (flag) {
    //删除的是末尾数据，size直接减一即可
    if (point + 1 == size) {
      alarm_ctrl->size --;
      display_alarm_info();
      return;
    }
    //删除的数据不止末尾数据，则需将删除的数据后面的数据全部前移一位
    alarm_ctrl->size --;
    for (i = point; i < size; i++) {
      alarm_ctrl->info[i].time = alarm_ctrl->info[i+1].time;
      alarm_ctrl->info[i].process_status = alarm_ctrl->info[i+1].process_status;
    }
    display_alarm_info();
  }
}

/*获取要设置的闹钟信息*/
uint8_t get_willalarm_info(uint32_t current_time, alarm_info_t *alarm_info)
{
  alarm_ctrl_t *alarm_ctrl = &alarmctrl;
  uint8_t i, flag = 0;
  if (0 == alarm_ctrl->size) {
    return 1;
  } else {
    //由于我们的闹钟数组是从小到大顺序排列，通过我们当前的时间去和闹钟数组时间对比
    //找到第一个比当前时间大的闹钟，我们设置即可，若未找到说明我们当前时间太大，那么我们选择数据开始的闹钟设置即可
    //如 数组 10 20 50 当前时间为18，那下个闹钟设置即是20，当前时间80，那么下个闹钟就是10
    for (i = 0; i < alarm_ctrl->size; i++) {
      if (current_time < alarm_ctrl->info[i].time) {
        flag = 1;
        alarm_info->time = alarm_ctrl->info[i].time;
        alarm_info->process_status = alarm_ctrl->info[i].process_status;
        break;
      }
    }
    if (0 == flag) {
      alarm_info->time = alarm_ctrl->info[0].time;
      alarm_info->process_status = alarm_ctrl->info[0].process_status;
    }
  }
  return 0;
}

/*业务处理*/
void alarm_handle_service()
{
  static int k = 0;
  alarm_ctrl_t *alarm_ctrl = &alarmctrl;
  printf("size = %d, setstatus = %d\r\n", alarm_ctrl->size, alarm_ctrl->set_status);
  //判断rtc是否同步，在STM32中，内置一个rtc，但当我们完全断电，rtc时间清掉，我们需重新同步
  uint8_t rtc_sync_status  = 1;
  if (!rtc_sync_status) {
    return;
  }
  //判断是否有闹钟需要设置
  if (0 == alarm_ctrl->size) {
    printf("not have alarm need set.....\r\n");
    return;
  }
  //判断闹钟是否设置
  if (ALARM_NOSET == alarm_ctrl->set_status) {
    //闹钟未设置
    //获取当前时间，设置闹钟
    uint32_t current_time;
    //current_time = get_local_time();//由于测试，此处不获取，自己赋值
    if (0 == k) {
      current_time = 5;
    } else if (1 == k) {
      current_time = 88;
    } else if (2 == k) {
      current_time = 300;
    } else {
      current_time = 1000;
      k = 0;
    }
    k ++;
    alarm_info_t alarm_info;
    uint8_t ret = get_willalarm_info(current_time, &alarm_info);
    if (0 == ret) {
      printf("set alarm %d\r\n", alarm_info.time);
      alarm_ctrl->process_status = alarm_info.process_status;
      alarm_ctrl->set_status = ALARM_SET;
    }
  } else {
    if (0 == get_alarm_clock()) {
      return;
    }
    //闹钟响，处理
    printf("alarm is clock....\r\n");
    switch (alarm_ctrl->process_status) {
      case PRINT_PEACE: {
        printf("Peace.........\r\n");
        break;
      }
      case PRINT_LOVE: {
        printf("Love.........\r\n");
        break;
      }
      case PRINT_HAVE_FUN: {
        printf("Have Fun.........\r\n");
        break;
      }
      default: {
        break;
      }
    }
    alarm_ctrl->set_status = ALARM_NOSET;
  }
}

int main()
{
  alarm_ctrl_init(&alarmctrl);

  alarm_info_t alarm_info;
  alarm_info.time =  100;
  alarm_info.process_status = PRINT_LOVE;
  insert_alarm_info(&alarmctrl, &alarm_info);

  alarm_info.time =  10;
  alarm_info.process_status = PRINT_PEACE;
  insert_alarm_info(&alarmctrl, &alarm_info);

  alarm_info.time =  500;
  alarm_info.process_status = PRINT_HAVE_FUN;
  insert_alarm_info(&alarmctrl, &alarm_info);

  alarm_info.time =  1000;
  alarm_info.process_status = PRINT_HAVE_FUN;
  insert_alarm_info(&alarmctrl, &alarm_info);

  delete_alarm_info(&alarmctrl, 1000);

  while (1) {
    alarm_handle_service();
    sleep(10);
  }
  return 0;
}

```



### 代码输出结果

​	这里是代码运行后的部分结果如下：

```
sky@ubuntu:~$ ./alarm_test
show alarm info: 
 100 
show alarm info: 
 10  100 
show alarm info: 
 10  100  500 
show alarm info: 
 10  100  500  1000 
show alarm info: 
 10  100  500 
size = 3, setstatus = 0
set alarm 10
size = 3, setstatus = 1
alarm is clock....
Peace.........
size = 3, setstatus = 0
set alarm 100
size = 3, setstatus = 1
alarm is clock....
Love.........
size = 3, setstatus = 0
set alarm 500
size = 3, setstatus = 1
alarm is clock....
Have Fun.........
size = 3, setstatus = 0
set alarm 10
size = 3, setstatus = 1
alarm is clock....
Peace.........
size = 3, setstatus = 0
set alarm 100

```