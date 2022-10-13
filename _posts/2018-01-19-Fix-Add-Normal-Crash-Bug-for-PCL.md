---
layout: post
section-type: post
title: PCL-- Bug Fix -- Crash when using addPointCloudNormals
category: engineering
tags: ["backend"]
---

## Problem

After compile [PCLVisualizer](http://pointclouds.org/documentation/tutorials/pcl_visualizer.php) in the following environment

1. MacOS 10.13.1 (17B1003)
2. PCL 1.8.0
3. VTK 8.0.0
4. cmake 3.10.0
5. Apple LLVM version 9.0.0 (clang-900.0.39.2) x86_64-apple-darwin17.2.0

The program will crash while it tries to run addPointCloudNormals as below

```C++
// Line92
viewer->addPointCloudNormals<pcl::PointXYZRGB, pcl::Normal> (cloud, normals, 10, 0.05, "normals");
// Line 169
viewer->addPointCloudNormals<pcl::PointXYZRGB, pcl::Normal> (cloud, normals1, 10, 0.05, "normals1", v1);
// Line 170
viewer->addPointCloudNormals<pcl::PointXYZRGB, pcl::Normal> (cloud, normals2, 10, 0.05, "normals2", v2);
```

In Windows the memory dump will be something like this, the error will reported as BAD ACCESS Ssegmentation fault which is not useful at all.

```
   pcltest.exe!vtkOStrStreamWrapper::str()  + 0xae bytes  C++
   pcltest.exe!vtkDemandDrivenPipeline::ExecuteDataObject()  + 0x42 bytes  C++
   pcltest.exe!vtkDemandDrivenPipeline::ProcessRequest()  + 0xab bytes  C++
   pcltest.exe!vtkStreamingDemandDrivenPipeline::ProcessRequest()  + 0x430 bytes  C++
   pcltest.exe!vtkExecutive::GetOutputInformation()  + 0x29 bytes  C++
   pcltest.exe!vtkDemandDrivenPipeline::ProcessRequest()  + 0x6e bytes  C++
   pcltest.exe!vtkStreamingDemandDrivenPipeline::ProcessRequest()  + 0x430 bytes  C++
   pcltest.exe!vtkDemandDrivenPipeline::UpdateDataObject()  + 0x81 bytes  C++
   pcltest.exe!vtkDemandDrivenPipeline::UpdateInformation()  + 0x23 bytes  C++
   pcltest.exe!vtkStreamingDemandDrivenPipeline::Update()  + 0xd bytes  C++
   pcltest.exe!vtkExecutive::Update()  + 0x20 bytes  C++
   pcltest.exe!vtkMapper::GetBounds()  + 0x70 bytes  C++
   pcltest.exe!vtkActor::GetBounds()  + 0x120 bytes  C++
   pcl_visualization_debug.dll!vtkFrustumCoverageCuller::Cull()  + 0x14c bytes
   pcl_visualization_debug.dll!vtkRenderer::AllocateTime()  + 0xba bytes
   pcl_visualization_debug.dll!vtkRenderer::Render()  + 0x703 bytes
   pcl_visualization_debug.dll!vtkRendererCollection::Render()  + 0xf3 bytes
   pcl_visualization_debug.dll!vtkRenderWindow::DoStereoRender()  + 0xd3 bytes
   pcl_visualization_debug.dll!vtkRenderWindow::DoFDRender()  + 0x462 bytes
   pcl_visualization_debug.dll!vtkRenderWindow::DoAARender()  + 0x5be bytes
   pcl_visualization_debug.dll!vtkRenderWindow::Render()  + 0x70a bytes
   pcl_visualization_debug.dll!vtkRenderWindowInteractor::Render()  + 0x44 bytes
   pcl_visualization_debug.dll!vtkHandleMessage2()  + 0xeb bytes
   pcl_visualization_debug.dll!vtkHandleMessage()  + 0xa9 bytes
   user32.dll!7e368734()
   [Frames below may be incorrect and/or missing, no symbols loaded for user32.dll]
   user32.dll!7e368816()
   user32.dll!7e378ea0()
   user32.dll!7e378eec()
   ntdll.dll!7c90e473()
   user32.dll!7e3694d2()
   user32.dll!7e378f10()
   user32.dll!7e3696c7()
   pcl_visualization_debug.dll!vtkWin32RenderWindowInteractor::Start()  + 0xa4 bytes
   pcl_visualization_debug.dll!pcl::visualization::PCLVisualizer::spinOnce(int time=100, bool force_redraw=false)  Line 449 + 0xae bytes  C++
```

The related two discussion can be found at [PCL User Mainling list](http://www.pcl-users.org/PCLVisualizer-crashes-after-addPointCloudNormals-td4029900.html) and [#Issue 1691](https://github.com/PointCloudLibrary/pcl/issues/1601)

## Cause

The new version PCL(1.8.0) is release by using OPENGL2 as its rendering backend. To be specific, VTK_RENDERING_BACKEND is OpenGL2 while compling PCL with VTK. VTK is a viualization engine which is used by pcl visualization as its engine.

## Solution

1.Download [VTK release source code](https://github.com/Kitware/VTK/releases) 8.0.0

2.While configuring VTK compiling options, choose OpenGL instead of OpenGL2

![image](/img/VTKCMake.png)

3.Make compile and make install for VTK

4.Download [PCL release source code](https://github.com/PointCloudLibrary/pcl/releases) 1.8.1

5.While configuring PCL compling options, choose the VTK we compiled by choosing OpenGL

![image](/img/PCLCmake.png)

6.Make compile and make install for PCL

7.While compling PCLVisualizer, choose the PCL and VTK we compiled just now.

![image](/img/VisualizationCmake.png)
