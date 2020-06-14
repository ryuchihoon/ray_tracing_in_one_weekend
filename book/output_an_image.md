# 2 출력 이미지

## 2.1 PPM 이미지 포맷

여러분들이 무엇인가를 렌더링하고 나서 렌더링 된 이미지를 볼 수 있는 방법이 필요합니다. 가장 직관적인 방법은 렌더링 결과를 파일로 저장하는것 입니다. 파일로 저장할 때 가장 어려운것은 어떤 포맷으로 저장할지 결정하는것 입니다. 왜냐하면 많은 이미지 포맷들이 복잡하기 때문입니다. 그래서 저는 항상 가장 간단한 ppm 파일을 사용합니다. ppm 포맷은 위키피디아에 굉장히 잘 설명되어 있습니다.

![](https://raytracing.github.io/images/img.ppm-example.jpg)

위키피디아와 동일한 로그를 출력하는 C++ 코드를 작성해봅시다:

```cpp
#include <iostream>

int main() {
    const int image_width = 256;
    const int image_height = 256;

    std::cout << "P3\n" << image_width << ' ' << image_height << "\n255\n";

    for (int j = image_height-1; j >= 0; --j) {
        for (int i = 0; i < image_width; ++i) {
            auto r = double(i) / (image_width-1);
            auto g = double(j) / (image_height-1);
            auto b = 0.25;

            int ir = static_cast<int>(255.999 * r);
            int ig = static_cast<int>(255.999 * g);
            int ib = static_cast<int>(255.999 * b);

            std::cout << ir << ' ' << ig << ' ' << ib << '\n';
        }
    }
}
```

코드에서 몇몇 부분을 주목해야합니다:

1. 픽셀들은 왼쪽에서 오른쪽으로 행에 저장됩니다.

2. 행들은 위에서 아래로 저장됩니다.

3. 관례에 따르면 빨강/초록/파랑 각각의 값들은 [0.0, 1.0] 사이의 값을 가지고 있습니다. 후반부에 HDR(High Dynamic Range)를 지원할 때는 이 범위보다 더 넓은 범위를 쓰지만, 출력 이미지에 저장할 때는 톤 맵핑을 사용해서 [0.0, 1.0] 사이의 값으로 변경합니다. 그러므로 이 부분에 해당하는 코드는 계속 유지될 것입니다.

4. 빨강색은 왼쪽에서 오른쪽으로 가면서 검은색에서 빨강색으로 변하게 되고 초록색은 아래에서 위로 가면서 검은색에서 초록색으로 변하게 됩니다. 빨강과 초록이 섞이면 노랑색이 되기 때문에 오른쪽 상단 부분은 노란색을 짐작할 수 있습니다.

## 2.2 이미지 파일 생성하기

프로그램의 출력을 이미지 파일로 저장해야 합니다. 전형적으로 커맨드 라인에 > 리디렉션 연산자를 사용함으로써 출력을 이미지로 저장할 수 있습니다. 아례 예시를 보시기 바랍니다.

```
build\Release\inOneWeekend.exe > image.ppm
```

위는 윈도우에서 출력을 이미지로 저장하는 예시입니다. 맥이나 리눅스는 아래와 같습니다.


```
build/inOneWeekend > image.ppm
```

저장된 이미지 파일을 열면(맥에선 ToyViewer로 볼 수 있습니다. 여러분들이 자주 사용하는 뷰어로 보셔도 되고 구글 "ppm viewer"로 볼 수 있습니다) 아래와 같은 결과를 볼 수 있습니다.

![](https://raytracing.github.io/images/img.first-ppm-image.png)

야호! 이것이 바로 그래픽스의 "hello world" 입니다. 이미지가 위처럼 보이지 않는다면, 출력 이미지를 텍스터 에디터로 여시고 안에 내용을 확인해보시길 바랍니다. 파일은 반드시 아래와 같은 내용으로 채워져 있어야 합니다.

```
P3
256 256
255
0 255 63
1 255 63
2 255 63
3 255 63
4 255 63
5 255 63
6 255 63
7 255 63
8 255 63
9 255 63
...
```

위와 다르다면 부수적인 새줄 문자나 다른 문자들로 인해 이미지 리더가 파일을 제대로 못해 발생하는 문제입니다.

PPM 말고 다른 이미지 포맷들을 사용하고 싶으시다면 ```stb_image.h```를 사용해보시길 바랍니다. 이 라이브러리는 헤더만 필요하기 때문에 사용하기 굉장히 편리합니다. https://github.com/nothings/stb 에서 다운받으실 수 있습니다.

## 2.3 진행 및 완료 출력하기

레이 트레이싱에 대해 배우기 전에 진행 상황을 알려주는 로그를 출력합시다. 이 로그는 렌더링이 길어질 때 진행상황을 볼 수 있는 아주 유용한 방법입니다. 그리고 무한 루프나 다른 문제에 의해서 프로그램이 멈추는 문제를 발견하는데도 도움이 됩니다.

프로그램이 표준 출력 스트림(```std::cout```)을 이용해서 이미지를 출력하기 때문에 표준 출력 스트림은 사용하지 않고 에러 출력 스트림(```std::cerr```)을 사용합니다.

```cpp
for (int j = image_height-1; j >= 0; --j) {
    std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;
    for (int i = 0; i < image_width; ++i) {
        auto r = double(i) / (image_width-1);
        auto g = double(j) / (image_height-1);
        auto b = 0.25;

        int ir = static_cast<int>(255.999 * r);
        int ig = static_cast<int>(255.999 * g);
        int ib = static_cast<int>(255.999 * b);

        std::cout << ir << ' ' << ig << ' ' << ib << '\n';
    }
}
std::cerr << "\nDone.\n";
```
 