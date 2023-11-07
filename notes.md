# PBRT分析

## 命令行控制
```bash
E:\Volume\pbrt\pbrt-v4\build\Release> ./pbrt E:\Volume\pbrt\pbrt-v4-scenes\bunny-cloud\bunny-cloud.pbrt --outfile E:\aaa.png --spp 1
```

## VS调试
解决方案cmd文件夹下pbrt_exe。

右键，属性-调试-命令参数 无需开头的./pbrt

在intergrators.cpp的render中渲染

ParallelFor2D渲染了大部分工作
进parallel.cpp
parallel只是一个并行框架，实际是用被调用的func，也就是integrators中的parallelFor2D参数中的那个函数


StartPixelSample在samplers.h中

RayIntegrator::EvaluatePixelSample算采样

## 代码细节

lu是什么？波长？如何作用
lambda又是什么

GetCameraSample获取像素中心位置（及控制是否需要jitter）

GenerateRayDifferential？
没懂
```c++
// Scale camera ray differentials based on image sampling rate
Float rayDiffScale =
    std::max<Float>(.125f, 1 / std::sqrt((Float)sampler.SamplesPerPixel()));
```


    小tip：
    漫反射贴图等非计算机计算出来的图像是非线性的，sRGB是非线性的，人眼对暗部更敏感

    输入一张贴图，这张贴图如果是线性贴图（通过计算机计算出来的纹理，例如法线和光照，Mask/AO贴图，因为他们只储存数据，那就可以直接输入，如果是sRGB空间下输出的（例如ps），需要先转换到线性空间中，这步的名字叫做Remove Gamma Correction，Unity中是勾选贴图中的sRGB（Color Texture）选项，如果引擎里没有这种选项的话，我们就需要自己进行这步操作，需要pow2.2得到真实的亮度，移动端游戏中，多为输出Tex值*Tex值。

    在线性空间中进行shader运算，输出前再gamma矫正回来

    https://zhuanlan.zhihu.com/p/53086060


重点在Li:
进VolPathIntegrator::Li
```c++
SampledSpectrum VolPathIntegrator::Li(RayDifferential ray, SampledWavelengths &lambda,
                                   Sampler sampler, ScratchBuffer &scratchBuffer,
                                   VisibleSurface *visibleSurf) const 
```
SampledSpectrum

contribution疑似加进film里，
```c++
// Add camera ray's contribution to image
camera.GetFilm().AddSample(pPixel, L, lambda, &visibleSurface, cameraSample.filterWeight);
```



GPU:

没进
```c++
template <typename ConcreteSampler>
void WavefrontPathIntegrator::GenerateCameraRays(int y0, Transform movingFromCamera,
                                                 int sampleIndex)
```

samples.cpp: template <typename ConcreteSampler> void WavefrontPathIntegrator::GenerateRaySamples(int wavefrontDepth, int sampleIndex) 



camera的initMetadata是在设置透视矩阵等

class Camera : public TaggedPointer<PerspectiveCamera, OrthographicCamera,
                                    SphericalCamera, RealisticCamera> 
class Film : public TaggedPointer<RGBFilm, GBufferFilm, SpectralFilm>

而对于film.addSample这一函数只有gbufferFilm类有实现，
因此推断taggedpointer是涵盖了括号中所有类型的，也就是一起在运行

但三种film的writeImage实现完全相同



## 数据存储
在.pbrt这个场景文件中把film改成gbuffer即可存下gbufferFilm的相关数据（具体含义可见https://www.pbrt.org/users-guide-v4#images），但albedo等都没有体积的数据

从GBufferFilm::AddSample中看到p(position),n(normal)等都只在visibleSurface上有效