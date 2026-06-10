# ERA5-
#运用python进行nc数据读取与绘图，并制作相关视频
#%%
#import netCDF4
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import glob
import os
import cv2
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import matplotlib as mpl                    # 导入matplotlib库，用于图形和颜色管理

from matplotlib.colors import BoundaryNorm 
from netCDF4 import Dataset, num2date
import xarray as xr
nc1 = Dataset('./data_stream-oper_stepType-instant.nc') 
nc2 = Dataset('./data_stream-oper_stepType-accum.nc')
for i in range(0,120,1):
    
    print(nc2.variables.keys())
    msl = np.array(nc1.variables['msl'][i, :, :])/100
    time1 = np.array(num2date(nc1.variables['valid_time'][:], units=nc1.variables['valid_time'].units))
    lat1 = np.array(nc1.variables['latitude'][:])
    lon1 = np.array(nc1.variables['longitude'][:])
    u10 = np.array(nc1.variables['u10'][i, :, :])
    v10 = np.array(nc1.variables['v10'][i, :, :])
    precipitation = np.array(nc2.variables['tp'][i, :, :])*1000
    print(precipitation.min(), precipitation.max())
    ##np.meshgrid(lon1, lat1)
    lon2d, lat2d = np.meshgrid(lon1, lat1)
    print(time1.shape)
    print(lat1.shape)
    print(lon1.shape)

    fig, ax = plt.subplots(
    1,2,
    figsize=(16,9),
    subplot_kw={"projection": ccrs.PlateCarree()}
)

    # 统一底图（只画一次）
    ax[0].add_feature(cfeature.COASTLINE.with_scale("50m"), linewidth=0.8)
    ax[0].add_feature(cfeature.BORDERS.with_scale("50m"), linewidth=0.5)
    ax[0].add_feature(cfeature.LAND.with_scale("50m"), alpha=0.2)
    ax[0].set_extent([-30, 8, 47, 62], crs=ccrs.PlateCarree())
    skip =10  # 选择每隔多少个点画一个风羽
    ax[0].barbs(
        lon2d[::skip, ::skip],
        lat2d[::skip, ::skip],
        u10[::skip, ::skip],
        v10[::skip, ::skip],
        barb_increments=dict(
            half=2,    # 短线 = 2
            full=4,    # 长线 = 4
            flag=20 
            ) ,  # 三角旗 = 20
        transform=ccrs.PlateCarree(),
        length=6,
        color ='purple',
        linewidth=0.5
    )
    gl = ax[0].gridlines(
    crs=ccrs.PlateCarree(),
    draw_labels=True,
    linewidth=0.6,
    linestyle="--",
    alpha=0.7
)

    gl.top_labels = False
    gl.right_labels = False
    cs = ax[0].contour(
    lon2d,
    lat2d,
    msl,
    levels=np.arange(960, 1044, 4),
    colors="blue",
    linewidths=1.0,
    transform=ccrs.PlateCarree()
)
    ax[0].set_title(
        f"MLP and 10 m Wind\n"
        f"{time1[i].strftime('%Y-%m-%d %H:%M UTC')}"
    )
    ax[0].clabel(cs, fmt="%d", fontsize=8)
    ax[1].add_feature(cfeature.COASTLINE.with_scale("50m"), linewidth=0.8)
    ax[1].add_feature(cfeature.BORDERS.with_scale("50m"), linewidth=0.5)
    ax[1].add_feature(cfeature.LAND.with_scale("50m"), alpha=0.2)
    ax[1].set_extent([-8, 2, 48, 52], crs=ccrs.PlateCarree())
    levels = [0, 0.01, 1, 2, 4, 6, 8, 10, 20, 50]
    colors = [
                "#FFFFFF",      # 0-0.01:白色
                "#C1FEBD",      # 0.01-1:浅绿色
                "#A4F293",      # 1-2:绿色
                "#38BB3A",      # 2-4:深绿色
                "#5FB7FC",      # 4-6:浅蓝色
                "#0001FC",      # 6-8:蓝色
                "#5F9E9B",      # 8-10:青色
                "#FD00FD",      # 10-20:紫色
                "#F93F41",      # 20-50:红色
            ]
    cmp = mpl.colors.ListedColormap(colors, "indexed")        # 创建颜色映射
        # cmp.N 返回 colors 列表的长度，即颜色的数量。clip:是否对超出边界的数据值进行裁剪。
    norm = BoundaryNorm(levels, cmp.N, clip=True)                   # 创建颜色归一化对象
    ax[1].contourf(
    lon2d,
    lat2d,
    precipitation,
    levels=levels,
    cmap=cmp,
    norm=norm,
    transform=ccrs.PlateCarree(),
    alpha=0.8
)
    skip =3  # 选择每隔多少个点画一个风羽
    ax[1].barbs(
        lon2d[::skip, ::skip],
        lat2d[::skip, ::skip],
        u10[::skip, ::skip],
        v10[::skip, ::skip],
        barb_increments=dict(
            half=2,    # 短线 = 2
            full=4,    # 长线 = 4
            flag=20 
            ) ,  # 三角旗 = 20
        transform=ccrs.PlateCarree(),
        length=8,
        color ='purple',
        linewidth=0.5
    )
    gl = ax[1].gridlines(
    crs=ccrs.PlateCarree(),
    draw_labels=True,
    linewidth=0.6,
    linestyle="--",
    alpha=0.7
)

    gl.top_labels = False
    gl.right_labels = False
   
    ax[1].set_title(
        f"Precipitation 1h and 10m Wind\n"
        f"{time1[i].strftime('%Y-%m-%d %H:%M UTC')}"
    )
    cbar =fig.colorbar(
    mpl.cm.ScalarMappable(norm=norm, cmap=cmp),
    ax=ax[1],
    orientation="vertical")   
    cbar.set_label("1-hour precipitation (mm)", fontsize=12)
    ax[0].set_aspect("auto")
    ax[1].set_aspect("auto")
    fig.savefig(f"dday_{i:03d}.png", dpi=600, bbox_inches="tight")
###############
import os
import re
import glob
import cv2


# 图片所在文件夹
image_dir = "."

# 输出视频
output_video = "./ERA5_weather.mp4"

# 视频帧率
fps = 2


def natural_sort_key(path):
    """
    自然排序：
    image_2.png 会排在 image_10.png 前面
    """
    filename = os.path.basename(path)

    return [
        int(part) if part.isdigit() else part.lower()
        for part in re.split(r"(\d+)", filename)
    ]


# 读取图片列表
image_files = glob.glob(
    os.path.join(image_dir, "*.png")
)

image_files = sorted(
    image_files,
    key=natural_sort_key
)

if len(image_files) == 0:
    raise FileNotFoundError(
        f"文件夹中没有找到 PNG 图片：{image_dir}"
    )

print("图片数量：", len(image_files))
print("第一张：", image_files[0])
print("最后一张：", image_files[-1])


# 读取第一张图片，确定视频尺寸
first_image = cv2.imread(image_files[0])

if first_image is None:
    raise RuntimeError(
        f"无法读取第一张图片：{image_files[0]}"
    )

height, width = first_image.shape[:2]

print("视频尺寸：", width, height)


# MP4 编码器
fourcc = cv2.VideoWriter_fourcc(*"mp4v")

video_writer = cv2.VideoWriter(
    output_video,
    fourcc,
    fps,
    (width, height)
)

if not video_writer.isOpened():
    raise RuntimeError(
        "视频创建失败，可能是当前环境不支持 mp4v 编码"
    )


# 逐张写入
for i, filename in enumerate(image_files, start=1):

    image = cv2.imread(filename)

    if image is None:
        print("跳过无法读取的图片：", filename)
        continue

    # 确保所有图片尺寸一致
    if image.shape[:2] != (height, width):
        image = cv2.resize(
            image,
            (width, height),
            interpolation=cv2.INTER_AREA
        )

    video_writer.write(image)

    print(
        f"{i}/{len(image_files)} "
        f"{os.path.basename(filename)}"
    )


video_writer.release()

print("视频生成完成：", output_video)
