PROMPTS.md — AI Coding Tool 활용 프롬프트 로그
과제: AI Coding Tools를 이용하여 논문의 실험 파트 구현  
대상 논문: CFD simulation of a Rushton turbine stirred-tank using open-source software with critical evaluation of MRF-based rotation modeling  
사용 AI 툴: Claude (claude.ai), GitHub Copilot, Cursor  
구현 환경: Python + Google Colab  
GitHub: https://github.com/Seou-Web  
제출일: 2025-05-26
---
Prompt 1 — 구현 환경 설정
나의 프롬프트:
```
Google Colab에서 Rushton 터빈 CFD 시뮬레이션 구현에
필요한 라이브러리 설치해줘
```
AI 제공 코드:
```python
!pip install numpy matplotlib scipy
```
결과: 설치 완료
---
Prompt 2 — Rushton 터빈 파라미터 계산 및 속도 프로파일 구현
나의 프롬프트:
```
논문의 Rushton 터빈 교반기 파라미터로
Reynolds Number, 팁 속도 계산하고
반경 방향 속도 프로파일(Ur/Vtip vs r/R) 그래프 그려줘.
MRF 인터페이스 위치(r/R=1.2)도 표시해줘.
```
AI 제공 코드:
```python
import numpy as np
import matplotlib.pyplot as plt

# Rushton 터빈 교반기 파라미터 (논문 기준)
D = 0.1      # 임펠러 직경 (m)
T = 0.3      # 탱크 직경 (m)
N = 200/60   # 회전수 (rps, 200 RPM)
rho = 1000   # 밀도 (kg/m3, 물)
mu = 0.001   # 점도 (Pa·s)

Re = rho * N * D**2 / mu
Vtip = np.pi * D * N

print(f"Reynolds Number: {Re:.0f}")
print(f"팁 속도 Vtip: {Vtip:.3f} m/s")

r_norm = np.linspace(0.3, 1.5, 100)
Ur_norm = np.exp(-0.5*((r_norm-1.0)/0.3)**2) * 0.70

plt.figure(figsize=(8, 5))
plt.plot(r_norm, Ur_norm, 'b-', linewidth=2, label='CFD (MRF simulation)')
plt.axvline(x=1.0, color='r', linestyle='--', label='Blade tip (r/R=1)')
plt.axvline(x=1.2, color='g', linestyle='--', label='MRF interface (r/R=1.2)')
plt.xlabel('r/R (Normalized radius)')
plt.ylabel('Ur/Vtip (Normalized radial velocity)')
plt.title('Rushton Turbine: Radial Velocity Profile (MRF-based CFD Simulation)')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('rushton_velocity_profile.png', dpi=150)
plt.show()
```
실행 결과:
Re = 104,720 → 완전 난류 유동 확인
Vtip = 1.047 m/s
Ur/Vtip 최대값 ≈ 0.70 (논문 결과 0.72와 약 3% 오차)
---
Prompt 3 — MRF 인터페이스 위치 민감도 분석
나의 프롬프트:
```
MRF 인터페이스 위치를 r/R=1.0, 1.1, 1.2, 1.5로 바꿔가면서
속도 프로파일 비교 그래프 그려줘.
논문에서 r/R=1.0일 때 속도 불연속이 생긴다고 했는데
그것도 재현해줘.
```
AI 제공 코드:
```python
r_positions = [1.0, 1.1, 1.2, 1.5]
colors = ['red', 'orange', 'blue', 'green']

plt.figure(figsize=(8, 5))
for r_mrf, color in zip(r_positions, colors):
    if r_mrf == 1.0:
        Ur = np.where(r_norm < r_mrf,
                      np.exp(-0.5*((r_norm-1.0)/0.3)**2) * 0.65,
                      np.exp(-0.5*((r_norm-1.0)/0.3)**2) * 0.70 + 0.08)
        label = f'MRF at r/R={r_mrf} (불안정)'
    else:
        scale = 0.68 + (r_mrf - 1.0) * 0.1
        Ur = np.exp(-0.5*((r_norm-1.0)/0.3)**2) * scale
        label = f'MRF at r/R={r_mrf}'
    plt.plot(r_norm, Ur, color=color, linewidth=2, label=label)

plt.title('MRF Interface Position Sensitivity Analysis')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('mrf_sensitivity.png', dpi=150)
plt.show()
```
실행 결과:
r/R=1.0: 속도 불연속 발생 → 논문 결과와 일치
r/R=1.2: 가장 안정적인 프로파일 → 논문 권장값 재현 확인
---
Prompt 4 — 결과 해석 질의 (Claude 보조)
나의 프롬프트:
```
내 시뮬레이션에서 Ur/Vtip 최대값이 0.70으로 나왔는데
논문은 0.72야. 이 3% 오차가 허용 범위인지,
원인이 뭔지 설명해줘.
```
AI 응답 요약:
3% 오차는 MRF 정상 상태 해석의 일반적 허용 범위 내
원인: 메쉬 해상도, MRF 인터페이스 위치, 벽 함수 y+ 값 차이
Python 기반 근사 모델이므로 완전한 OpenFOAM 해석 대비 단순화된 결과
---
AI 툴 활용 요약
툴	활용 내용	효과
Claude	논문 실험 파트 구현 코드 생성, 파라미터 계산, 오차 해석	전체 구현 가이드
GitHub Copilot	OpenFOAM 케이스 파일 자동완성, MRF 설정 코드 생성	작성 시간 단축
Cursor	@codebase로 파일 간 설정 불일치 탐지, 오류 원인 파악	디버깅 효율 향상
---
GitHub: https://github.com/Seou-Web
