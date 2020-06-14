# 3 벡터3 클래스

대부분의 그리팩스 프로그램들은 지오메트릭 벡터들과 컬러들을 저장하는 몇개의 클래스들을 가지고 있습니다. 몇몇 시스템에서는 이런 벡터들이 4D(지오메트리를 위한 3D + 호모지니어스 좌표계 그리고 색상을 위한 RGB + 알파 투명도 채널)입니다. 이 책에서는 3차원 좌표계면 충분합니다. 우리는 ```vec3``` 클래스를 컬러, 위치, 방향, 오프셋 등 다양한곳에 사용합니다. 어떤 사람들은 ```vec3```를 여러곳에 사용하는것을 못마땅하게 생각합니다. 왜냐하면 색상과 위치를 더하는것과 같은 말도 안되는 연산이 실행되기 때문입니다. 물론 좋은 의견이지만 우리는 명백하게 잘못된 경우를 제외하곤 항상 코드를 덜 작성하는 방향을 선택할 것입니다. 그래서 우리는 ```vec3```의 두 개의 별칭: ```point3```와 ```color```로도 선언합니다 그러므로 두 타입이 단지 ```vec3```의 별칭이기 때문에 ```point3```를 넘겨야하는 함수에 ```color```를 넘긴다 할지라도 어떤 경고 메세지도 확인할 수 없습니다. 그러므로 반드시 사용하려는 목적을 할 때만 사용해야합니다.

## 3.1 변수와 함수

아래는 ```vec3``` 클래스의 윗 부분입니다:

```cpp
#ifndef VEC3_H
#define VEC3_H

#include <cmath>
#include <iostream>

using std::sqrt;

class vec3 {
    public:
        vec3() : e{0,0,0} {}
        vec3(double e0, double e1, double e2) : e{e0, e1, e2} {}

        double x() const { return e[0]; }
        double y() const { return e[1]; }
        double z() const { return e[2]; }

        vec3 operator-() const { return vec3(-e[0], -e[1], -e[2]); }
        double operator[](int i) const { return e[i]; }
        double& operator[](int i) { return e[i]; }

        vec3& operator+=(const vec3 &v) {
            e[0] += v.e[0];
            e[1] += v.e[1];
            e[2] += v.e[2];
            return *this;
        }

        vec3& operator*=(const double t) {
            e[0] *= t;
            e[1] *= t;
            e[2] *= t;
            return *this;
        }

        vec3& operator/=(const double t) {
            return *this *= 1/t;
        }

        double length() const {
            return sqrt(length_squared());
        }

        double length_squared() const {
            return e[0]*e[0] + e[1]*e[1] + e[2]*e[2];
        }

    public:
        double e[3];
};

// vec3의 타입 별명들
using point3 = vec3;   // 3D point
using color = vec3;    // RGB color

#endif
```

여기에서 ```double```을 사용하지만 다른 레이 트레이서는 ```float```을 사용하기도 합니다. 두 타입 모두 사용해도 무방하며 개인에 취향에 맞게 선택하시면 됩니다.

## 3.2 벡터3 유틸리티 함수

헤더 파일의 두번째 부분은 벡터 유틸리티 함수입니다:

```cpp
// vec3 유틸리티 함수들

inline std::ostream& operator<<(std::ostream &out, const vec3 &v) {
    return out << v.e[0] << ' ' << v.e[1] << ' ' << v.e[2];
}

inline vec3 operator+(const vec3 &u, const vec3 &v) {
    return vec3(u.e[0] + v.e[0], u.e[1] + v.e[1], u.e[2] + v.e[2]);
}

inline vec3 operator-(const vec3 &u, const vec3 &v) {
    return vec3(u.e[0] - v.e[0], u.e[1] - v.e[1], u.e[2] - v.e[2]);
}

inline vec3 operator*(const vec3 &u, const vec3 &v) {
    return vec3(u.e[0] * v.e[0], u.e[1] * v.e[1], u.e[2] * v.e[2]);
}

inline vec3 operator*(double t, const vec3 &v) {
    return vec3(t*v.e[0], t*v.e[1], t*v.e[2]);
}

inline vec3 operator*(const vec3 &v, double t) {
    return t * v;
}

inline vec3 operator/(vec3 v, double t) {
    return (1/t) * v;
}

inline double dot(const vec3 &u, const vec3 &v) {
    return u.e[0] * v.e[0]
         + u.e[1] * v.e[1]
         + u.e[2] * v.e[2];
}

inline vec3 cross(const vec3 &u, const vec3 &v) {
    return vec3(u.e[1] * v.e[2] - u.e[2] * v.e[1],
                u.e[2] * v.e[0] - u.e[0] * v.e[2],
                u.e[0] * v.e[1] - u.e[1] * v.e[0]);
}

inline vec3 unit_vector(vec3 v) {
    return v / v.length();
}
```

## 3.3 색상 유틸리티 함수

새롭게 정의한 ```vec3``` 클래스를 사용하면서, 한 픽셀의 색상을 표준 출력 스트림으로 출력하는 유틸리티 함수를 하나 만들것 입니다.

```cpp
#ifndef COLOR_H
#define COLOR_H

#include "vec3.h"

#include <iostream>

void write_color(std::ostream &out, color pixel_color) {
    // [0, 255]로 변환된 각 컴포넌트의 값을 출력합니다.
    out << static_cast<int>(255.999 * pixel_color.x()) << ' '
        << static_cast<int>(255.999 * pixel_color.y()) << ' '
        << static_cast<int>(255.999 * pixel_color.z()) << '\n';
}

#endif
```

이제 메인 함수를 아래와 같이 변경할 수 있습니다.

```cpp
#include "color.h"
#include "vec3.h"

#include <iostream>

int main() {
    const int image_width = 256;
    const int image_height = 256;

    std::cout << "P3\n" << image_width << ' ' << image_height << "\n255\n";

    for (int j = image_height-1; j >= 0; --j) {
        std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;
        for (int i = 0; i < image_width; ++i) {
            color pixel_color(double(i)/(image_width-1), double(j)/(image_height-1), 0.25);
            write_color(std::cout, pixel_color);
        }
    }

    std::cerr << "\nDone.\n";
}
```
