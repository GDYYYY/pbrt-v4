# PBRT分析

## 命令行控制：
```bash
E:\Volume\pbrt\pbrt-v4\build\Release> ./pbrt E:\Volume\pbrt\pbrt-v4-scenes\bunny-cloud\bunny-cloud.pbrt --outfile E:\aaa.png --spp 1
```

## VS调试：
解决方案cmd文件夹下pbrt_exe。

右键，属性-调试-命令参数 无需开头的./pbrt

在intergrators.cpp的render中渲染

ParallelFor2D渲染了大部分工作
进parallel.cpp
parallel只是一个并行框架，实际是用被调用的func，也就是integrators中的parallelFor2D参数中的那个函数


StartPixelSample在samplers.h中

RayIntegrator::EvaluatePixelSample算采样