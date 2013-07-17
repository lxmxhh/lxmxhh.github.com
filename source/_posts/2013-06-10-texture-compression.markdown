---
layout: post
title: "texture compression"
date: 2013-06-10 23:00
comments: true
categories: 图形编程
---
Texture compression support

Texture compression can significantly increase the performance of your OpenGL application by reducing memory requirements and making more efficient use of memory bandwidth. <!--more-->The Android framework provides support for the ETC1 compression format as a standard feature, including a ETC1Util utility class and the etc1tool compression tool (located in the Android SDK at `<sdk>/tools/`). For an example of an Android application that uses texture compression, see the CompressedTextureActivity code sample.

The ETC format is supported by most Android devices, but it not guarranteed to be available. To check if the ETC1 format is supported on a device, call the `ETC1Util.isETC1Supported()` method.

Note: The ETC1 texture compression format does not support textures with an alpha channel. If your application requires textures with an alpha channel, you should investigate other texture compression formats available on your target devices.

Beyond the ETC1 format, Android devices have varied support for texture compression based on their GPU chipsets and OpenGL implementations. You should investigate texture compression support on the devices you are are targeting to determine what compression types your application should support. In order to determine what texture formats are supported on a given device, you must query the device and review the OpenGL extension names, which identify what texture compression formats (and other OpenGL features) are supported by the device. Some commonly supported texture compression formats are as follows:

ATITC (ATC) - ATI texture compression (ATITC or ATC) is available on a wide variety of devices and supports fixed rate compression for RGB textures with and without an alpha channel. This format may be represented by several OpenGL extension names, for example:
GL_AMD_compressed_ATC_texture  
GL_ATI_texture_compression_atitc  
PVRTC - PowerVR texture compression (PVRTC) is available on a wide variety of devices and supports 2-bit and 4-bit per pixel textures with or without an alpha channel. This format is represented by the following OpenGL extension name:  
GL_IMG_texture_compression_pvrtc  
S3TC (DXTn/DXTC) - S3 texture compression (S3TC) has several format variations (DXT1 to DXT5) and is less widely available. The format supports RGB textures with 4-bit alpha or 8-bit alpha channels. This format may be represented by several OpenGL extension names, for example:  
GL_OES_texture_compression_S3TC  
GL_EXT_texture_compression_s3tc  
GL_EXT_texture_compression_dxt1  
GL_EXT_texture_compression_dxt3  
GL_EXT_texture_compression_dxt5  
3DC - 3DC texture compression (3DC) is a less widely available format that supports RGB textures with an an alpha channel. This format is represented by the following OpenGL extension name:
GL_AMD_compressed_3DC_texture  
Warning: These texture compression formats are not supported on all devices. Support for these formats can vary by manufacturer and device. For information on how to determine what texture compression formats are on a particular device, see the next section.

Note: Once you decide which texture compression formats your application will support, make sure you declare them in your manifest using `<supports-gl-texture>` . Using this declaration enables filtering by external services such as Google Play, so that your app is installed only on devices that support the formats your app requires. For details, see OpenGL manifest declarations.