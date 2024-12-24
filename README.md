# panfrost
panfrost integration for rockchip platform

## rk356x/rk3576 Mali-G52
### 1. Linux kernel部分修改：
#### 1.1 修改gpu dts配置
#### rk356x:
```
    kernel 5.10: arch/arm64/boot/dts/rockchip/rk3568.dtsi
    kernel 6.1: arch/arm64/boot/dts/rockchip/rk356x.dtsi

    gpu: gpu@fde60000 {
            compatible = "rockchip,rk3568-mali", "arm,mali-bifrost";
            reg = <0x0 0xfde60000 0x0 0x4000>;
            interrupts = <GIC_SPI 40 IRQ_TYPE_LEVEL_HIGH>,
                         <GIC_SPI 41 IRQ_TYPE_LEVEL_HIGH>,
                         <GIC_SPI 39 IRQ_TYPE_LEVEL_HIGH>;
            interrupt-names = "job", "mmu", "gpu";
            clocks = <&scmi_clk 1>, <&cru CLK_GPU>;
            clock-names = "gpu", "bus";
            operating-points-v2 = <&gpu_opp_table>;
            power-domains = <&power RK3568_PD_GPU>;                                                                                                          
            status = "disabled";
    };

    gpu_opp_table: opp-table2 {
        compatible = "operating-points-v2";

        mbist-vmin = <825000 900000 950000>;
        nvmem-cells = <&gpu_leakage>, <&core_pvtm>, <&mbist_vmin>;
        nvmem-cell-names = "leakage", "pvtm", "mbist-vmin";
        rockchip,pvtm-voltage-sel = <
            0        84000   0
            84001    91000   1
            91001    100000  2
        >;
        rockchip,pvtm-ch = <0 5>;

        opp-200000000 {
            opp-hz = /bits/ 64 <200000000>;
            opp-microvolt = <850000>;
            opp-microvolt-L0 = <850000>;
            opp-microvolt-L1 = <825000>;
            opp-microvolt-L2 = <825000>;
        };   
        opp-300000000 {
            opp-hz = /bits/ 64 <300000000>;
            opp-microvolt = <850000>;
            opp-microvolt-L0 = <850000>;
            opp-microvolt-L1 = <825000>;
            opp-microvolt-L2 = <825000>;
        };   
        opp-400000000 {
            opp-hz = /bits/ 64 <400000000>;
            opp-microvolt = <850000>;
            opp-microvolt-L0 = <850000>;
            opp-microvolt-L1 = <825000>;
            opp-microvolt-L2 = <825000>;
        };   
        opp-600000000 {
            opp-supported-hw = <0xfb 0xffff>;
            opp-hz = /bits/ 64 <600000000>;
            opp-microvolt = <875000>;
            opp-microvolt-L0 = <875000>;
            opp-microvolt-L1 = <825000>;
            opp-microvolt-L2 = <825000>;
        };
        opp-700000000 {
            opp-hz = /bits/ 64 <700000000>;                                                                                                                                              
            opp-microvolt = <950000>;
            opp-microvolt-L0 = <950000>;
            opp-microvolt-L1 = <900000>;
            opp-microvolt-L2 = <850000>;
        };   
        opp-800000000 {
            opp-hz = /bits/ 64 <800000000>;
            opp-microvolt = <1000000>;
            opp-microvolt-L0 = <1000000>;
            opp-microvolt-L1 = <950000>;
            opp-microvolt-L2 = <900000>;
        };   

    }; 
```
#### rk3576:
```
    kernel 6.1: arch/arm64/boot/dts/rockchip/rk3576.dtsi
    gpu: gpu@27800000 {
            compatible = "arm,mali-bifrost";
            reg = <0x0 0x27800000 0x0 0x20000>;
            interrupts = <GIC_SPI 347 IRQ_TYPE_LEVEL_HIGH>,
                         <GIC_SPI 348 IRQ_TYPE_LEVEL_HIGH>,
                         <GIC_SPI 349 IRQ_TYPE_LEVEL_HIGH>;
            interrupt-names = "job", "mmu", "gpu";
            clocks = <&scmi_clk CLK_GPU>, <&cru CLK_GPU>;
            clock-names = "gpu", "bus";
			assigned-clocks = <&cru CLK_GPU>;
			assigned-clock-rates = <800000000>;
            power-domains = <&power RK3576_PD_GPU>;
			dynamic-power-coefficient = <1625>;
            status = "disabled";
    };
```

#### 1.2 修改arch/arm64/configs/rockchip_linux_defconfig，启用panfrost驱动
```
CONFIG_DRM_PANFROST=y
```
#### 1.3 重新编译linux kernel，烧写boot.img，从启动信息中确认panfrost驱动加载情况：

```
[    3.267980] panfrost fde60000.gpu: clock rate = 594000000
[    3.268023] panfrost fde60000.gpu: bus_clock rate = 200000000
[    3.269270] panfrost fde60000.gpu: mali-g52 id 0x7402 major 0x1 minor 0x0 status 0x0
[    3.269299] panfrost fde60000.gpu: features: 00000000,13de77ff, issues: 00000000,00000400
[    3.269322] panfrost fde60000.gpu: Features: L2:0x07110206 Shader:0x00000002 Tiler:0x00000209 Mem:0x1 MMU:0x00002823 AS:0xff JS:0x7
[    3.269341] panfrost fde60000.gpu: shader_present=0x1 l2_present=0x1
[    3.271985] [drm] Initialized panfrost 1.1.0 20180908 for fde60000.gpu on minor 0
```

 
### 2. 安装mesa
安装必要的软件包：

sudo apt install -y x11proto-core-dev xtrans-dev libpixman-1-dev libx11-dev libxkbfile-dev libxfont-dev nettle-dev systemd libudev-dev libdrm-dev libepoxy-dev mesa-common-dev nettle-dev libudev-dev libgbm-dev libepoxy-dev libxshmfence-dev libxcb-xinput-dev

#### 2.1 编译 libdrm:
```
git clone https://gitlab.freedesktop.org/mesa/drm.git
cd drm
meson build --prefix=/usr/local
sudo ninja -C build install
```
#### 2.2 编译 mesa:
```
git clone https://gitlab.freedesktop.org/mesa/mesa.git
cd mesa
git checkout -f mesa-24.3.2 -b 24.3.2
# 如需wayland后端支持，设置-Dplatforms=x11,wayland
meson build -Dvulkan-drivers=panfrost -Dgallium-drivers=panfrost -Dplatforms=x11 -Dglx=auto -Dprefix=/usr/local
sudo ninja -C build install
```
#### 2.3 编译 xserver:
```
git clone https://gitlab.freedesktop.org/xorg/xserver.git
cd xserver
git checkout -f xorg-server-21.1.14 -b xserver-21.1.14
meson build -D prefix=/usr/local -D glamor=true
sudo ninja -C build install
sudo mkdir -p /var/local/log
```
#### 2.4 编译 libinput:
```
git clone https://gitlab.freedesktop.org/xorg/driver/xf86-input-libinput.git
cd xf86-input-libinput 
meson build
sudo ninja -C build install
```
### 3.	系统配置信息修改
#### 3.1 修改/usr/bin/X, 禁用libdrm-cursor。并修改Xorg启动路径
```
#export LD_PRELOAD=/usr/lib/aarch64-linux-gnu/libdrm-cursor.so.1
basedir=/usr/local/bin
```
#### 3.2 修改/etc/ld.so.conf.d/00-aarch64-mali.conf文件，更新系统链接路径，优先加载mesa驱动
```
/usr/local/lib
```
修改完成后，执行以下命令更新共享库缓存，并重启机器：
```
ldconfig
```
 
### 4.	安装相关工具，查看图形系统信息
#### 4.1.查看glx支持情况
```
apt install -y mesa-utils
glxinfo -B
display: :0  screen: 0
direct rendering: Yes
Extended renderer info (GLX_MESA_query_renderer):
    Vendor: Mesa (0xffffffff)
    Device: Mali-G52 r1 (Panfrost) (0xffffffff)
    Version: 23.0.0
    Accelerated: yes
    Video memory: 1960MB
    Unified memory: yes
    Preferred profile: core (0x1)
    Max core profile version: 3.1
    Max compat profile version: 3.1
    Max GLES1 profile version: 1.1
    Max GLES[23] profile version: 3.1
OpenGL vendor string: Mesa
OpenGL renderer string: Mali-G52 r1 (Panfrost)
OpenGL core profile version string: 3.1 Mesa 23.0.0 (git-bbf142b8de)
OpenGL core profile shading language version string: 1.40
OpenGL core profile context flags: (none)

OpenGL version string: 3.1 Mesa 23.0.0 (git-bbf142b8de)
OpenGL shading language version string: 1.40
OpenGL context flags: (none)

OpenGL ES profile version string: OpenGL ES 3.1 Mesa 23.0.0 (git-bbf142b8de)
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.10
```
#### 4.2.查看egl支持情况
```
apt install -y mesa-utils-extra
eglinfo
EGL client extensions string:
    EGL_EXT_client_extensions EGL_EXT_device_base
    EGL_EXT_device_enumeration EGL_EXT_device_query EGL_EXT_platform_base
    EGL_KHR_client_get_all_proc_addresses EGL_KHR_debug
    EGL_EXT_platform_device EGL_EXT_platform_x11 EGL_KHR_platform_x11
    EGL_EXT_platform_xcb EGL_MESA_platform_gbm EGL_KHR_platform_gbm
    EGL_MESA_platform_surfaceless
EGL API version: 1.4
EGL vendor string: Mesa Project
EGL version string: 1.4
EGL client APIs: OpenGL OpenGL_ES 
EGL extensions string:
    EGL_ANDROID_blob_cache EGL_ANDROID_native_fence_sync
    EGL_EXT_buffer_age EGL_EXT_image_dma_buf_import
    EGL_EXT_image_dma_buf_import_modifiers EGL_KHR_cl_event2
    EGL_KHR_config_attribs EGL_KHR_context_flush_control
    EGL_KHR_create_context EGL_KHR_create_context_no_error
    EGL_KHR_fence_sync EGL_KHR_get_all_proc_addresses
    EGL_KHR_gl_colorspace EGL_KHR_gl_renderbuffer_image
    EGL_KHR_gl_texture_2D_image EGL_KHR_gl_texture_3D_image
    EGL_KHR_gl_texture_cubemap_image EGL_KHR_image EGL_KHR_image_base
    EGL_KHR_image_pixmap EGL_KHR_no_config_context EGL_KHR_partial_update
    EGL_KHR_reusable_sync EGL_KHR_surfaceless_context
    EGL_EXT_pixel_format_float EGL_KHR_wait_sync
    EGL_MESA_configless_context EGL_MESA_drm_image
    EGL_MESA_image_dma_buf_export EGL_MESA_query_driver
```
