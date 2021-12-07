---
title: 'Django给子组件配置路由'
date: '2018-11-28'
spoiler: ''
---

  # 1. App路由配置 urls.py文件
```python
from django.urls import path, re_path, include

from . import views

urlpatterns = [
    path('', views.index, name="index"),
    path('home/',views.home,name='home'),
    path('uav/', views.uav, name='uav'),
    path('uav/hardware/', views.uav_hardware, name='hardware'),
    path('uav/message/', views.uav_message, name='message')
]
```
说明：这里的name就是在路由跳转时对应的name。**注意urlpatterns配置成数组。**

# 2.路由对应的视图(html) views.py
```python
def index(request):
    uavs = refresh_uav().values()
    return render(request, "home.html", {'uavs':uavs})


def home(request):
    uavs = refresh_uav().values()
    return render(request, "home.html", {'uavs': uavs})


def uav(requset):
    uav = None
    profiles = ['DeviceCode', 'UavType', 'tty_usbserial', 'ModuleId', 
        'OperationState', 'OperationStatus1', 'OperationStatus2', 'HardwareStatus', 
        'Request', 'MissionId', 'UavTime', 'Online', 'Latitude', 'Longitude']
    if 'deviceCode' in requset.GET.keys():
        deviceCode = requset.GET['deviceCode']
        uav = find_uav_by_device_code(deviceCode)
        print(f"uav is {uav}")
        # print(f"find uav is :{uav.device_code}")
    return render(requset, "uav.html", {'uav': json.dumps(uav), 'profiles': json.dumps(profiles)})


def uav_hardware(request):
    hardware = ['OperationState', 'OperationStatus1', 'OperationStatus2', 'Request', 
    'HardwareStatus', 'GPS Module', 'Different GPS', 'IMU', 
    'Distance Sensor', 'Height Sensor', 'Vision Module', 'Gateway Moudle', 'R/C Module', 
    'Power Module', 'Spray Module', 'Pump Module', 'Pump Motor', 'Flight Motor', 
    'Firmware compatibility issue', 'Poor GPS', 'Current Sensor', 'Camera Damage', 
    'Compass Module', 'FC GPS Fault', 'FC Poor GPS', 'Power Module Failed', 
    'Flight Controller', 'CPU']
    return render(request, "uav_hardware.html", {'hardware': json.dumps(hardware)})


def uav_message(request):
    return render(request, "uav_message.html")
```

说明：可传参数。参数必须为json字符串。

# 3.配置跳转
在uav.html中

```html
    <ul class="nav nav-tabs">
        <li class="nav-item">
            <a class="nav-link active" href="">Profile</a>
        </li>
        <li class="nav-item">
            <a class="nav-link" href="{% url 'message' %}">Message</a>
        </li>
        <li class="nav-item">
            <a class="nav-link" href="{% url 'hardware' %}">Hardware</a>
        </li>
    </ul>
```
说明：这里的路由匹配跳转要填写 在urls.py文件中路由映射对应的name。
  