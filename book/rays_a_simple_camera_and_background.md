# 4 레이, 간단한 카메라, 배경

## 4.1 레이 클래스

모든 레이 트레이서가 공통적으로 가지고 있는 한 가지는 레이 클레스와 레이를 따라가면서 어떤 색이 보이는지 계산하는 것입니다. 레이 함수를 𝐏(𝑡)=𝐀+𝑡𝐛로 생각해 봅시다. 여기서 𝐏는 3차원에서 한 선에 있는 한 점입니다. 𝐀는 레이 시작점이고 𝐛는 레이 방향입니다. 레이 파라미터 𝑡는 실수(코드에선 double)입니다. 레이 파라미터 𝑡에 다른 값을 대입하면 𝐏(𝑡)의 결과인 점은 레이를 따라 이동하게 됩니다. 𝑡에 음수도 사용할 수 있습니다. 그러므로 우리는 3차원 선 위에서 원하는 곳으로 마음대로 갈수 있습니다. 보통 𝑡의 범위를 양수로 한정해서 𝐀의 앞으로만 이동하는데 이것을 하프 라인 또는 레이라고 부릅니다.

![](https://raytracing.github.io/images/fig.lerp.jpg)

코드에서 함수 𝐏(𝑡)는 ```ray::at(t)```에 구현되어 있습니다.

```cpp
#ifndef RAY_H
#define RAY_H

#include "vec3.h"

class ray {
    public:
        ray() {}
        ray(const point3& origin, const vec3& direction)
            : orig(origin), dir(direction)
        {}

        point3 origin() const  { return orig; }
        vec3 direction() const { return dir; }

        point3 at(double t) const {
            return orig + t*dir;
        }

    public:
        point3 orig;
        vec3 dir;
};

#endif
```

## 4.2 레이를 장면으로 쏘기

이제 레이 트레이서를 만들 준비가 됬습니다. 핵심은 레이 트레이서가 레이를 픽셀로 보내고 레이가 나아가는 방향에 어떤 색이 보이는지 계산하는 것입니다. 최종 색은 몇 단계에 걸쳐서 계산됩니다. (1) 눈으로부터 픽셀로 향하는 레이를 계산합니다. (2) 레이와 교차하는 물체들을 찾아냅니다. (3) 교차 지점에서의 색을 계산합니다. 항상 처음 레이 트레이서를 개발할때는 가장 간단한 카메라의 구현부터 시작합니다. 그리고 배경 색(간단한 그라디언트)을 반환하는 간단한 ```ray_color(ray)```함수를 구현합니다.

저는 정사각형 이미지를 쓰는 경우 디버깅을 하는데 있어서 종종 문제를 겪습니다. 왜냐하면 x와 y를 너무 자주 바꿔쓰기 때문입니다. 그래서 정사각형이 아닌 이미지를 사용할 예정입니다. 지금부터 이미지에서 일반적으로 사용되는 16:9 종횡비를 사용하도록 하겠습니다.

렌더링될 이미지의 픽셀 크기를 변경하는것 말고도 씬 레이들이 통과할 버츄얼 뷰포트도 설정해야합니다. 표준 정사각형 픽셀 간격에서는 뷰포트의 종횡비가 렌더링될 이미지와 동일해야합니다. 여기서는 높이가 2 단위인 뷰포를 사용하도록 하겠습니다. 또한 프로젝션 평면과 프로젝션 지점 사이의 거리가 1 단위가 되는 거리도 설정해야합니다. 이것은 "초점 거리"라고 하며 나중에 배울 "초점 거리"와 혼동하지 마시기 바랍니다.

저는 눈(카메라로 생각해도 됩니다)을 (0, 0, 0)에 놓을 것입니다. 저는 y축을 위로 향하고 x축을 오른쪽으로 향하는 오른손 좌표계를 사용할 것입니다. 오른손 좌표계의 관례에 의해서 화면은 z축의 음의 방향에 존재하게 됩니다. 저는 화면을 좌측 하단부터 횡단할 것입니다 그리고 화면을 따라 레이의 엔드포인트를 이동하기 위해 두 개의 오프셋 벡터를 사용해서 화면을 횡단할 것입니다. 주목할 것은 레이 방향이 단위 벡터로 만들지 않을 것입니다. 왜냐하면 단위 벡터로 만드는 것이 코드를 단순하게 만들지 않는다고 생각하고 성능이 약간 더 빠르기 때문입니다.

![](https://raytracing.github.io/images/fig.cam-geom.jpg)

아래 코드에서 레이 ```r```은 대충 픽셀의 중심을 통과합니다(여기서 정확하게 픽셀의 중심을 통과할 필요는 없습니다 왜냐하면 나중에 안티 앨리어싱을 적용할 것이기 때문입니다):

```cpp
#include "ray.h"

#include <iostream>

color ray_color(const ray& r) {
    vec3 unit_direction = unit_vector(r.direction());
    auto t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*color(1.0, 1.0, 1.0) + t*color(0.5, 0.7, 1.0);
}

int main() {
    const auto aspect_ratio = 16.0 / 9.0;
    const int image_width = 384;
    const int image_height = static_cast<int>(image_width / aspect_ratio);

    std::cout << "P3\n" << image_width << " " << image_height << "\n255\n";

    auto viewport_height = 2.0;
    auto viewport_width = aspect_ratio * viewport_height;
    auto focal_length = 1.0;

    auto origin = point3(0, 0, 0);
    auto horizontal = vec3(viewport_width, 0, 0);
    auto vertical = vec3(0, viewport_height, 0);
    auto lower_left_corner = origin - horizontal/2 - vertical/2 - vec3(0, 0, focal_length);

    for (int j = image_height-1; j >= 0; --j) {
        std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;
        for (int i = 0; i < image_width; ++i) {
            auto u = double(i) / (image_width-1);
            auto v = double(j) / (image_height-1);
            ray r(origin, lower_left_corner + u*horizontal + v*vertical - origin);
            color pixel_color = ray_color(r);
            write_color(std::cout, pixel_color);
        }
    }

    std::cerr << "\nDone.\n";
}
```

```ray_color(ray)``` 함수는 흰색과 파란색을 선형적으로 블렌딩합니다. 얼만큼 블렌딩할 것인가는 레이 방향을 단위 벡터(그러므로 −1.0 <. 𝑦 <. 1.0)로 변경한 다음에 y좌표의 높이에 의해서 결정됩니다. 벡터를 정규화한 뒤에 y좌표의 높이를 사용하기 때문에 세로 그라디언트 외에도 가로 그라디언트도 볼 수 있습니다.

그 다음에 이 값을 0.0 ≤ 𝑡 ≤ 1.0을 변환하는 그래픽스에서 자주 사용하는 트릭을 사용했습니다. 저는 𝑡 = 1.0이면 파랜색이 출력되고 𝑡 = 0.0이면 하얀색이 출력되도록 만들었습니다. 그리고 𝑡가 0.0과 1.0 사이의 값일 경우엔 블렌딩이 되도록 처리하였습니다. 이러한 계산 방식을 선형 블렌딩, 리니어 인터폴레이션, lerp라고 부릅니다. lerp는 아래의 공식으로 표현됩니다.

blendedValue = (1 − 𝑡) ⋅ startValue + 𝑡 ⋅ endValue

이 공식에서 𝑡는 0에서 1로 변하게 됩니다. 위의 코드를 실행하시면 아래와 같은 결과를 얻을 수 있습니다.

![](https://raytracing.github.io/images/img.blue-to-white.png)
















