### I/O模拟SPI的驱动实现

#### 1. 数据结构

struct rt_spi_bit_ops主要定义了模拟SPI总线时序时需要的数据结构，这里便携模拟SPI底层驱动时需要适配这个结构

* data	: 一般用于关联GPIO操作端口、引脚、时钟等配置信息
* tog_sclk : SCLK脚电平翻转函数
* set_sclk : 配置SCLK脚电平状态函数
* set_mosi : 配置MOSI脚电平状态函数
* set_miso : 配置MISO脚电平状态函数
* get_sclk : 获取SCLK脚电平状态函数
* get_mosi : 获取MOSI脚电平状态函数
* get_miso : 获取MISO脚电平状态函数
* dir_mosi : 配置MOSI脚输入输出状态函数
* dir_miso : 配置MISO脚输入输出状态函数
* udelay : 微秒延时函数
* delay_us : 微秒延时时间，不用用户设定

```
struct rt_spi_bit_ops
{
    void *const data;            /* private data for lowlevel routines */
    void (*const tog_sclk)(void *data);
    void (*const set_sclk)(void *data, rt_int32_t state);
    void (*const set_mosi)(void *data, rt_int32_t state);
    void (*const set_miso)(void *data, rt_int32_t state);
    rt_int32_t (*const get_sclk)(void *data);
    rt_int32_t (*const get_mosi)(void *data);
    rt_int32_t (*const get_miso)(void *data);

    void (*const dir_mosi)(void *data, rt_int32_t state);
    void (*const dir_miso)(void *data, rt_int32_t state);

    void (*const udelay)(rt_uint32_t us);
    rt_uint32_t delay_us;  /* sclk、mosi and miso line delay */
};
```

struct rt_spi_bit_obj主要继承自struct rt_spi_bus，关联struct rt_spi_bit_ops底层操作适配和struct rt_spi_configuration总线配置记录

```
struct rt_spi_bit_obj
{
    struct rt_spi_bus bus;
    struct rt_spi_bit_ops *ops;
    struct rt_spi_configuration config;
};
```

#### 2. 注册接口

底层开发人员使用`rt_spi_bit_add_bus(obj, bus_name, ops)`接口将模拟总线注册到IO设备框架中，其原型为

```
rt_err_t rt_spi_bit_add_bus(struct rt_spi_bit_obj *obj, 
                            const char            *bus_name,
                            struct rt_spi_bit_ops *ops);
```
