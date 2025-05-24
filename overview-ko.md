# 프로젝트 목적

안드로이드에서 GStreamer를 사용 할 수 있도록, 안드로이드의 카메라 API를 GStreamer에 통합시키는 프로젝트입니다.

더 정확히 말하자면, GStreamer의 Plugins으로 편성하는 것 입니다. 또한, GStreamer에서 CameraBin 으로 이용하기 위해, Photography 인터페이스 및 CameraBin을 상속받도록 구현해야합니다.

현재 안드로이드 카메라 API 버전인 Camera2를 대상으로 버전업이 목적입니다.

크게 두가지 섹션으로 이해하면 따라가기에 쉬울 것입니다. 

1. 안드로이드 카메라 API

2. GStreamer에서의 카메라 제어

---

---

# 1. 안드로이드 카메라  API

## History

안드로이드 카메라 API의 버전이 올라가면서 구현이 달라집니다. [안드로이드 카메라 개발 문서](https://developer.android.com/media/camera/get-started-with-camera?hl=ko)를 참고해주세요.

### API 버전 목록

- CameraX
- Camera2
- Camera (deprecated)

### Camera (deprecated)

> 편의 및 혼란을 방지하기 위해 이하 Camera1 이라 부르겠습니다.

현재 [GStreamer에 적용](https://github.com/gstreamer101/gstreamer/blob/android/camera/subprojects/gst-plugins-bad/sys/androidmedia/gstahcsrc.c)되어있는 API입니다.

해당 안드로이드 카메라 API는 deprecated 되었고, Camera2로 대체되었습니다.

### Camera2 (**버전업 대상 API**)

**버전업 대상 API** 입니다. 

Camera1과는 다르게, C/C++ 라이브러리 (== 네이티브 라이브러리) 를 제공한다는 것이 주요 포인트입니다.

[C/C++ 라이브러리는 NDK](https://developer.android.com/ndk/reference/group/camera)를 통해 제공됩니다.

### CameraX

Camera2를 사용하기 쉽게 wrapping한 클래스입니다. 안드로이드에서 직접 카메라를 제하고 싶을때, 안드로이드 카메라의 하위 수준 접근을 하지 않는다면 Camera2가 아닌 CameraX를 사용하도록 권장합니다.

## 버전 업그레이드 특기 사항

### Camera1은 C/C++ 라이브러리가 제공되지 않았습니다.

따라서 Java 로 제공되는 인터페이스를 C/C++ 에서 사용할 수 있도록 JNI로 연동시키는 작업이 필요했습니다. 

안드로이드의 `android.hardware.Camera` Java API를 C 코드에서 사용할 수 있도록 JNI(Java Native Interface)를 통해 래핑 구현한 [기존 plugins 예시](https://github.com/gstreamer101/gstreamer/blob/android/camera/subprojects/gst-plugins-bad/sys/androidmedia/gst-android-hardware-camera.c)를 확인해보세요

### Camera2는 C/C++ 라이브러리가 제공됩니다

따라서 JNI로 연동시키는 작업이 필요없습니다. NDK에서 제공되는 인터페이스를 이용하여 바로 작업하면 됩니다!

### GStreamer 플러그인으로 구현시, 성능 관점 차이점

GStreamer는 C로 작성된 프로젝트입니다. 따라서 Camera1 API를 사용하기위해서는 JNI로 사용해야합니다. 하지만, 직접 JVM의 메모리를 참조할 수 없기 때문에 메모리 복사가 필요합니다. 

![Camera 메모리 처리 구조도](https://github.com/user-attachments/assets/103a03df-092b-4b01-9ef7-64bc158d7c1f)

> 출처: Justin Kim, *"A source element for Android Camera 2 NDK API (Lightning Talk)"*, GStreamer Conference 2017. [PDF 링크]([https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2017/Justin Kim - A source element for Android Camera 2 NDK API (Lightning Talk).pdf](https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2017/Justin%20Kim%20-%20A%20source%20element%20for%20Android%20Camera%202%20NDK%20API%20(Lightning%20Talk).pdf))

하지만, Camera2 버전부터는 C/C++ 라이브러리가 제공되기 때문에 직접 메모리 참조가 가능하므로 메모리복사가 필요없습니다. 따라서, 더 높은 성능을 낼 수 있게되었습니다. 

![Camera 메모리 처리 구조도](https://github.com/user-attachments/assets/13ac7576-2603-414d-afd0-8ced5279fbd3)

> 출처: Justin Kim, *"A source element for Android Camera 2 NDK API (Lightning Talk)"*, GStreamer Conference 2017. [PDF 링크]([https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2017/Justin Kim - A source element for Android Camera 2 NDK API (Lightning Talk).pdf](https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2017/Justin%20Kim%20-%20A%20source%20element%20for%20Android%20Camera%202%20NDK%20API%20(Lightning%20Talk).pdf))

---

## Camera2 기반의 예시 프로젝트 (Not maintained)

과거에 진행되었던, 참고할만한 좋은 [예시 프로젝트](https://gitlab.collabora.com/joykim/gst-plugins-bad/-/tree/32bda1596087c05fd67433f7c694d741d0e8c8e3/sys/androidndk)가 있어서 소개합니다.

예시 프로젝트를 보면, 안드로이드 카메라를 [C/C++ 라이브러리로 직접 사용](https://gitlab.collabora.com/joykim/gst-plugins-bad/-/blob/32bda1596087c05fd67433f7c694d741d0e8c8e3/sys/androidndk/ndk-internal.h)합니다

```c
#include <camera/NdkCameraDevice.h>
#include <camera/NdkCameraManager.h>
```

**Q**. 그 외에도, 파일이 많은데 무엇인가요?

**A**. 추가적인 작업을 편하게하기 위해 모듈화 하는 작업들 입니다

**ACameraManager_getCameraIdList 함수** 를 예시로 분석해보겠습니다. 

[**ACameraManager_getCameraIdList** - 안드로이드 개발 문서](https://developer.android.com/ndk/reference/group/camera#acameramanager_getcameraidlist)를 보면, 

> ACameraManager_getCameraIdList will allocate and return an [ACameraIdList](https://developer.android.com/ndk/reference/struct/a-camera-id-list#struct_a_camera_id_list). The caller must call [ACameraManager_deleteCameraIdList](https://developer.android.com/ndk/reference/group/camera#group___camera_1gab2fc4e660b80ff1af4e104b895868724) to free the memory

라고 기술 되어있습니다. 메모리 해제가 반드시 필요한 것이죠.

그러한 [일련의 작업을 모듈화](https://gitlab.collabora.com/joykim/gst-plugins-bad/-/blob/32bda1596087c05fd67433f7c694d741d0e8c8e3/sys/androidndk/ndk-camera-manager.c#L118) 한 것임을 확인 할 수 있습니다

```c
gboolean
ndk_camera_manager_get_camera_list (const NdkCameraManager * manager,
    GList ** list)
{
  ACameraIdList *camera_id_list = NULL;

  g_return_val_if_fail (NDK_IS_CAMERA_MANAGER (manager), FALSE);
  g_return_val_if_fail (manager->camera_manager != NULL, FALSE);

  if (ACameraManager_getCameraIdList (manager->camera_manager,
          &camera_id_list) == ACAMERA_OK) {
    int i = 0;
    for (; i < camera_id_list->numCameras; i++) {
      *list = g_list_append (*list, g_strdup (camera_id_list->cameraIds[i]));
    }
    ACameraManager_deleteCameraIdList (camera_id_list);
    return TRUE;
  }

  return FALSE;
}
```

NDK 를 이용하여 안드로이드 카메라 resource 를 직접 접근하는 것 까지 이해 되었을 것입니다.

---

---

# 2. GStreamer 에서의 카메라 제어

이제는 카메라를 GStreamer 에서 제어하는 플로우에 대해 알아봅니다.

크게 두 섹션으로 알아보겠습니다. 

1. 카메라 파라미터 제어를 위한 GstPhotography 인터페이스

2. 캡처 파이프라인 구성 및 스트림 제어를 위한 CameraBin 및 GstBaseCameraSrc 인터페이스

---

## 1. 카메라 파라미터 제어 - **GstPhotography**

**GstPhotography**는 `gst-plugins-bad` 모듈에서 제공하는 **불안정(unstable) GObject 인터페이스**로, 카메라의 **줌(ZOOM)**, **초점(FOCUS)**, **노출 보정(EV_COMPENSATION)**, **ISO 감도**, **화이트밸런스**, **샤프닝** 등 **디지털 이미지 캡처 파라미터**를 다룹니다.

예시 API (zoom 관련)

- `gst_photography_set_zoom()`
- `gst_photography_get_zoom()`

하지만, **GstPhotography**는 공식 웹 문서에선 지원되지 않습니다.

---

### 인터페이스 구현 방식

기존의 [ahcsrc - Camera1 element](https://github.com/gstreamer101/gstreamer/blob/android/camera/subprojects/gst-plugins-bad/sys/androidmedia/gstahcsrc.c#L71)를 바탕으로 인터페이스 구현 방식을 알아보겠습니다.

#### 1. `GstPhotography` 인터페이스 등록

[코드 링크](https://github.com/gstreamer101/gstreamer/blob/android/camera/subprojects/gst-plugins-bad/sys/androidmedia/gstahcsrc.c#L212)

```c
G_DEFINE_TYPE_WITH_CODE (
  GstAHCSrc,
  gst_ahc_src,
  GST_TYPE_PUSH_SRC,
  G_IMPLEMENT_INTERFACE (GST_TYPE_PHOTOGRAPHY,
                        gst_ahc_src_photography_init)
);
```

- 이 매크로는 `GstAHCSrc` 타입을 정의하면서, 동시에 **`GST_TYPE_PHOTOGRAPHY`** 인터페이스를 구현하도록 GObject 시스템에 등록합니다.
- `gst_ahc_src_photography_init` 함수가 인터페이스 초기화 콜백으로 사용됩니다.

#### 2. 인터페이스 초기화 및 메서드 바인딩

[코드 링크](https://github.com/gstreamer101/gstreamer/blob/android/camera/subprojects/gst-plugins-bad/sys/androidmedia/gstahcsrc.c#L237)

```c
static void
gst_ahc_src_photography_init (gpointer g_iface, gpointer iface_data)
{
  GstPhotographyInterface *iface = g_iface;

  iface->get_ev_compensation      = gst_ahc_src_get_ev_compensation;
  iface->set_ev_compensation      = gst_ahc_src_set_ev_compensation;
  iface->get_white_balance_mode   = gst_ahc_src_get_white_balance_mode;
  iface->set_white_balance_mode   = gst_ahc_src_set_white_balance_mode;
  iface->get_colour_tone_mode     = gst_ahc_src_get_colour_tone_mode;
  iface->set_colour_tone_mode     = gst_ahc_src_set_colour_tone_mode;
  iface->get_scene_mode           = gst_ahc_src_get_scene_mode;
  iface->set_scene_mode           = gst_ahc_src_set_scene_mode;
  iface->get_flash_mode           = gst_ahc_src_get_flash_mode;
  iface->set_flash_mode           = gst_ahc_src_set_flash_mode;
  iface->get_zoom                 = gst_ahc_src_get_zoom;
  iface->set_zoom                 = gst_ahc_src_set_zoom;
  iface->get_flicker_mode         = gst_ahc_src_get_flicker_mode;
  iface->set_flicker_mode         = gst_ahc_src_set_flicker_mode;
  iface->get_focus_mode           = gst_ahc_src_get_focus_mode;
  iface->set_focus_mode           = gst_ahc_src_set_focus_mode;
  iface->get_capabilities         = gst_ahc_src_get_capabilities;
  iface->set_autofocus            = gst_ahc_src_set_autofocus;
}
```

- `gst_ahc_src_photography_init`에서 `GstPhotographyInterface` 구조체의 각 함수 포인터를 **ahcsrc** 전용 구현(예: `gst_ahc_src_get_zoom`, `gst_ahc_src_set_flash_mode` 등)으로 채웁니다.
- 이로써 애플리케이션이 `gst_photography_set_zoom()` 등을 호출하면, 실제로 위에 바인딩된 메서드가 실행되어 안드로이 카메라 API를 통해 장치 설정을 변경합니다

#### 함수 구현 예시 `gst_ahc_src_get_zoom`

[코드 링크](https://github.com/gstreamer101/gstreamer/blob/android/camera/subprojects/gst-plugins-bad/sys/androidmedia/gstahcsrc.c#L1529)

```c
static gboolean
gst_ahc_src_set_zoom (GstPhotography * photo, gfloat zoom)
{
  GstAHCSrc *self = GST_AHC_SRC (photo);
  gboolean ret = FALSE;

  if (self->camera) {
    GstAHCParameters *params;

    params = gst_ah_camera_get_parameters (self->camera);
    if (params) {
      GList *zoom_ratios = gst_ahc_parameters_get_zoom_ratios (params);
      gint max_zoom = gst_ahc_parameters_get_max_zoom (params);
      gint zoom_idx = -1;

      if (zoom_ratios && g_list_length (zoom_ratios) == (max_zoom + 1)) {
        gint i;
        gint value = zoom * 100;

        for (i = 0; i < max_zoom + 1; i++) {
          gint zoom_value = GPOINTER_TO_INT (g_list_nth_data (zoom_ratios, i));

          if (value == zoom_value)
            zoom_idx = i;
        }
      }

      if (zoom_idx != -1) {
        if (self->smooth_zoom &&
            gst_ahc_parameters_is_smooth_zoom_supported (params)) {
          // First, we need to cancel any previous smooth zoom operation
          gst_ah_camera_stop_smooth_zoom (self->camera);
          ret = gst_ah_camera_start_smooth_zoom (self->camera, zoom_idx);
        } else {
          gst_ahc_parameters_set_zoom (params, zoom_idx);
          ret = gst_ah_camera_set_parameters (self->camera, params);
        }
      }

      gst_ahc_parameters_zoom_ratios_free (zoom_ratios);
      gst_ahc_parameters_free (params);
    }
  }

  return ret;
}
```

Camera2 API 기반의 구현에서는, JNI로 뚫어둔 함수 구현들이 C/C++ 라이브러리를 직접 사용하도록 바뀔 것 입니다

---

## **2. 캡처 파이프라인** 구성 및 스트림 제어 - CameraBin 및 GstBaseCameraSrc

카메라 “캡처 파이프라인 구성 및 스트림 제어” 계층은 **CameraBin** 이라는 고수준 요소와, 이를 구성하기 위한 추상 베이스 클래스인 **GstBaseCameraSrc** 로 이루어집니다.

[**CameraBin**](https://gstreamer.freedesktop.org/documentation/camerabin/camerabin.html?gi-language=c)은 정지 이미지·비디오 녹화·뷰파인더 기능을 한 번에 제공하는 컨테이너 요소이며, [**GstBaseCameraSrc**](https://gstreamer.freedesktop.org/documentation/basecamerabinsrc/element-basecamerasrc.html?gi-language=c)는 CameraBin 내부에서 실제 하드웨어 카메라 소스를 3개의 Ghost Pad(`vfsrc`, `imgsrc`, `vidsrc`) 로 노출하기 위해 상속받아 구현해야 하는 추상 클래스입니다.

우리의 프로젝트를 CameraBin으로 활용하려면 GstBaseCameraSrc를 상속하여 카메라 source element 를 구현해야 합니다.

---

### 인터페이스 구현 방식

// TODO

---

**Q.** pushsrc는 뭔가요. 기존의 프로젝트들에서는 pushsrc를 사용하던데요.

[pushsrc 사용 예시](https://github.com/gstreamer101/gstreamer/blob/android/camera/subprojects/gst-plugins-bad/sys/androidmedia/gstahcsrc.h#L29)

**A.** 간단하게 요약하자면, GstBaseCameraSrc 가 최신이고 인터페이스가 더 디테일해졌다고 보면 됩니다. 앞으로의 작업들은 CameraBin에 맞게 진행하면 됩니다
