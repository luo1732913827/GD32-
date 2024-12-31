非堵塞按键扫描的核心思想是将按键状态的检查和事件的触发分开进行，每次扫描仅对按键状态进行检查和更新，不会阻塞程序的其他操作。通过使用去抖动、状态机和事件触发机制，能够在保证响应速度的同时，避免按键操作对系统其他任务的干扰。
1. 按键初始化 (key_init)
任务：初始化按键相关硬件配置。
步骤：
启用 GPIO 时钟：rcu_periph_clock_enable(KEY_RCPU)
配置 GPIO 为输入模式：gpio_init(keys[i].gpio_periph, GPIO_MODE_IPU, GPIO_OSPEED_50MHZ, keys[i].pin)
2. 按键扫描函数 (key_scan)
任务：定期扫描按键状态，判断按键事件。
子模块：
按键电平读取：通过 GPIO 读取按键电平 (key_level = gpio_input_bit_get(...))
状态机处理：
KEY_RELEASED: 按键松开状态
按下：进入 KEY_PRESSED 状态
KEY_PRESSED: 按键按下状态
持续按下超过 KEY_LONG_TIME，进入 KEY_LONG_PRESSED 状态，触发长按事件
按键松开时，若按下时间小于 KEY_LONG_TIME，触发短按事件
KEY_LONG_PRESSED: 长按状态
如果按下时间超过 KEY_CONTINUE_TIME，触发长按事件并重置计数器
按键松开时，进入 KEY_RELEASED 状态并触发释放事件
KEY_CONTINUE: 连按状态
持续按下超过 KEY_CONTINUE_TIME，触发连按事件
3. 按键事件管理
key_get_event：获取按键事件
返回按键当前事件：KEY_EVENT_NONE, KEY_EVENT_SHORT, KEY_EVENT_LONG, KEY_EVENT_CONTINUE, KEY_EVENT_RELEASE
key_clear_event：清除按键事件标志
清除指定按键的事件
key_is_pressed：获取按键当前状态
判断按键是否被按下，返回 1 或 0
4. 按键事件类型
短按事件 (KEY_EVENT_SHORT)
按下时间小于 KEY_LONG_TIME，按键松开时触发
长按事件 (KEY_EVENT_LONG)
按下时间超过 KEY_LONG_TIME，触发长按事件
连按事件 (KEY_EVENT_CONTINUE)
持续按下超过 KEY_CONTINUE_TIME，触发连按事件
释放事件 (KEY_EVENT_RELEASE)
按键松开时触发
5. 定时器与定时任务
定时器初始化：
启用定时器时钟
配置定时器计时周期为 10ms
使能定时器中断
定时器中断处理：
定时器中断时调用 key_scan() 扫描按键


按键初始化：初始化 GPIO 和时钟，配置按键为输入模式（上拉）。
按键扫描函数：负责读取 GPIO 电平并根据不同的时间条件和状态进行状态机判断，触发不同的按键事件（短按、长按、连按、释放）。
按键事件管理：通过 key_get_event() 获取当前按键的事件，key_clear_event() 清除事件标志，以及 key_is_pressed() 获取按键当前状态。
按键事件类型：不同的按键时间区间触发不同的事件类型。
定时器与定时任务：通过定时器周期性调用 key_scan() 函数实现按键扫描和事件触发。
