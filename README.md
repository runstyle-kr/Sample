# 손글씨 숫자 인식 데모 (GitHub + Vercel 배포 테스트)

캔버스에 쓴 숫자를 MNIST 방식(28×28)으로 변환해 인식하는 정적 웹 앱입니다.
서버가 필요 없어서 Vercel 무료 플랜으로 바로 배포할 수 있습니다.

- **모델 없이도 동작**: `model.onnx`가 없으면 간단한 규칙 기반 데모 모드로 실행됩니다.
  배포 파이프라인(GitHub → Vercel)을 먼저 검증한 뒤, 나중에 모델만 추가하면 됩니다.
- **모델을 넣으면**: 학습한 모델을 `model.onnx` 이름으로 저장소 루트에 올리면
  페이지가 자동으로 감지해 실제 모델로 인식합니다.

## 1. 로컬에서 먼저 확인

```powershell
# 프로젝트 폴더에서 (아무 파이썬이나 가능)
python -m http.server 8000
# 브라우저에서 http://localhost:8000 열기
```

`index.html`을 더블클릭해도 됩니다(데모 모드는 동작, model.onnx 로드는 서버 필요).

## 2. GitHub에 올리기 

```powershell
git init
git add .
git commit -m "Add handwriting demo"
```

GitHub에서 새 저장소(예: `handwriting-demo`)를 만든 뒤:

```powershell
git remote add origin https://github.com/<내계정>/handwriting-demo.git
git branch -M main
git push -u origin main
```

## 3. Vercel 배포

1. https://vercel.com 접속 → GitHub 계정으로 로그인
2. **Add New → Project** → 방금 만든 저장소 **Import**
3. Framework Preset은 **Other** 그대로 두고 **Deploy**
4. 1분 안에 `https://handwriting-demo-xxxx.vercel.app` 주소가 생깁니다.

이후에는 `git push`만 하면 자동으로 재배포됩니다.

## 4. 실제 모델 연결 (PyTorch 예시)

학습이 끝난 모델을 ONNX로 변환:

```python
import torch

model.eval()
dummy = torch.randn(1, 1, 28, 28)
torch.onnx.export(model, dummy, "model.onnx",
                  input_names=["input"], output_names=["output"])
```

만들어진 `model.onnx`를 이 저장소 루트에 넣고 push하면 끝입니다.

주의: 모델이 입력을 0~1 값으로 받도록 학습했는지, 표준화((x−0.1307)/0.3081)된 값으로
받는지 확인하세요. 표준화 모델이면 `index.html`의 `normalize` 변수를 `true`로 바꾸세요.

## 폴더 구조

```
handwriting-demo/
├── index.html    # 앱 전체 (HTML + CSS + JS)
├── vercel.json   # 배포 설정 (선택)
├── .gitignore    # .venv, data 등 제외
└── model.onnx    # (선택) 학습한 모델 — 넣으면 자동 인식
```

`.venv/`, `data/MNIST/` 같은 학습용 파일은 `.gitignore`에 있어 올라가지 않습니다.
