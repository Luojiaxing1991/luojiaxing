+ 使能SCSI_LOGGING
+ 设置LOG level

# 使能SCSI_LOGGING
CONFIG_SCSI_LOGGING=y

# 设置LOG level

## 通过grub.cfg命令行指定
scsi_mod.scsi_logging_level=0x600

写0可以关闭IO打印

## 通过sysfs配置
echo 0x600 > /sys/module/scsi_mod/parameters/scsi_logging_level
