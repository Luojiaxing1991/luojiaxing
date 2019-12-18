# 概述
这个文件夹主要用来构建内核GIC模块与其ITS子模块的系统知识架构

# 文件夹层级
- gic_arch.md  //介绍GIC模块的整体框架
- irq_dest     //介绍外围中断源
   - lpi&msi.md
- its          //介绍ITS模块
   + init.md
   + table_for_its_dev.md  //介绍ITS转发中断所需要的各类表格
   + lpi.md    //介绍LPI中断的分配回收
   + irq       //介绍中断的申请，注入，释放
      - alloc.md
      - free.md
      - request.md
   + virt      //介绍ITS的虚拟化
- GIC_v4
      
