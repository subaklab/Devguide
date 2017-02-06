# Matrice 100

{% youtube %}https://www.youtube.com/watch?v=3OGs0ONemGc{% endyoutube %}

![Matrice 100](images/airframes/multicopter/matrice100/Matrice100.jpg)

## 부품 리스트

  * [DJI Matrice 100](http://store.dji.com/product/matrice-100) ESC, 모터, 프레임

## 모터 연결

![Connections](images/airframes/multicopter/matrice100/Wiring Diagram.jpg)

![Wiring Harness](images/airframes/multicopter/matrice100/WiringHarness.jpg)

![PWM Connections](images/airframes/multicopter/matrice100/PwmInput.jpg)

![Top](images/airframes/multicopter/matrice100/Top.jpg)

![Back](images/airframes/multicopter/matrice100/Back.jpg)

![No Stack](images/airframes/multicopter/matrice100/NoStack.jpg)

![No Top Deck](images/airframes/multicopter/matrice100/NoTopDeck.jpg)

| Output | Rate | Actuator |
| -- | -- | -- |
| MAIN1 | 400 Hz | Front right, CCW |
| MAIN2 | 400 Hz | Back left, CCW |
| MAIN3 | 400 Hz | Front left, CW |
| MAIN4 | 400 Hz | Back right, CW |
| AUX1 | 50 Hz | RC AUX1 |
| AUX2 | 50 Hz | RC AUX2 |
| AUX3 | 50 Hz | RC AUX3 |

## 파라미터

* 높은 throttle에서 inner loop은 기본 x quad gain을 사용하면 진동이 발생합니다. 낮은 throttle에서 gain을 더 높이면 더 나은 응답성을 제공합니다. throttle 기반의 일부 gain scheduling은 전반적인 응답성이 개선될 수 있고 이는 mc_att_control에서 구현할 수 있습니다. 우선 낮거나 높은 throttle에서 진동이 없도록 튜닝할 것이고 낮은 throttle에서 대역폭을 줄이도록 합니다.
	* MC_PITCHRATE_P: 0.05
	* MC_PITCHRATE_D: 0.001
* 배터리는 기본 3셀 대신에 6셀로
	* BAT_N_CELLS: 6
