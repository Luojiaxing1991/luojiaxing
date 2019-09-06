# ITS maintainer's kernel git repo
https://git.kernel.org/pub/scm/linux/kernel/git/maz/arm-platforms.git/  

# Debug ITS

我们可以在irq-gic-v3-its.c的文件开始加上  
#define DEBUG 1  
这样追加pr_debug这一部分的打印  

e.g
[    8.435707] hisi_sas_v3_hw 0000:74:02.0: Adding to iommu group 0  
[    8.461467] scsi host0: hisi_sas_v3_hw  
[    9.683463] ITS: alloc 9920:32  
[    9.686509] ITT 32 entries, 5 bits  
[    9.690044] ID:0 pID:9920 vID:23  
[    9.693263] ID:1 pID:9921 vID:24  
[    9.696480] ID:2 pID:9922 vID:25  
[    9.699696] ID:3 pID:9923 vID:26  
[    9.702911] ID:4 pID:9924 vID:27  
[    9.706128] ID:5 pID:9925 vID:28  
[    9.709344] ID:6 pID:9926 vID:29  
[    9.712560] ID:7 pID:9927 vID:30  
[    9.715776] ID:8 pID:9928 vID:31  
[    9.718990] ID:9 pID:9929 vID:32  
[    9.722207] ID:10 pID:9930 vID:33  
[    9.725510] ID:11 pID:9931 vID:34  
[    9.728813] ID:12 pID:9932 vID:35  
[    9.732116] ID:13 pID:9933 vID:36  
[    9.735419] ID:14 pID:9934 vID:37  
[    9.738721] ID:15 pID:9935 vID:38  
[    9.742024] ID:16 pID:9936 vID:39   


