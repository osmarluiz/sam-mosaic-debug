# SAM-Mosaic Debug - Memory Leak & CLI Hang Investigation

## Problema

O CLI do sam-mosaic trava em "Loading SAM2 model..." no PC2, mas funciona perfeitamente no PC1.
Curiosamente, chamar `build_sam2()` diretamente funciona no PC2.

---

## Configurações dos PCs

### PC1 (Funciona)

| Item | Valor |
|------|-------|
| GPU | NVIDIA GeForce RTX 4090 24GB |
| Driver NVIDIA | 560.94 |
| PyTorch | 2.6.0+cu124 |
| CUDA | 12.4 |
| cuDNN | 90100 |
| SAM2 | 1.1.0 |
| OS | Windows 11 Pro (26200.0) |
| Resultado | CLI funciona, 18 min para 10k×10k |

### PC2 (Trava)

| Item | Valor |
|------|-------|
| GPU | (preencher) |
| Driver NVIDIA | (preencher) |
| PyTorch | (preencher) |
| CUDA | (preencher) |
| cuDNN | (preencher) |
| SAM2 | (preencher) |
| OS | (preencher) |
| Resultado | CLI trava no load do modelo |

**Comandos para obter configuração:**
```powershell
nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv
conda activate ts_annotator
python -c "import torch; print(f'PyTorch: {torch.__version__}'); print(f'CUDA: {torch.version.cuda}'); print(f'cuDNN: {torch.backends.cudnn.version()}')"
pip show sam2 | Select-String 'Version'
[System.Environment]::OSVersion
```

---

## Hipóteses

### 1. OMP_NUM_THREADS=1 (Principal suspeito)

O `sam_mosaic/__init__.py` define `OMP_NUM_THREADS=1` no Windows para evitar memory leak do scikit-learn.
Isso pode conflitar com PyTorch/CUDA em certas configurações.

**Teste:**
```powershell
$env:OMP_NUM_THREADS='4'
$env:SAM_MOSAIC_DEBUG='1'
sam-mosaic input.tif output/ --checkpoint checkpoints/sam2.1_hiera_large.pt
```

### 2. Checkpoint corrompido

**Verificar:**
```powershell
dir checkpoints\sam2.1_hiera_large.pt
certutil -hashfile checkpoints\sam2.1_hiera_large.pt MD5
```

**Valores esperados:**
- Tamanho: 857 MB (898,921,545 bytes)
- MD5: `2b30654b6112c42a115563c638d238d9`

### 3. Diferença no ambiente conda

**Solução:** Usar environment.yml do repo para replicar ambiente exato:
```powershell
conda env create -f environment.yml
conda activate ts_annotator
```

### 4. Versão do Windows/WSL2

Se PC2 usa WSL2 diferente, pode haver conflito com CUDA.

---

## Debug Mode

O código foi atualizado com modo debug. Para ativar:

```powershell
$env:SAM_MOSAIC_DEBUG='1'
sam-mosaic input.tif output/ --checkpoint checkpoints/sam2.1_hiera_large.pt
```

**Output esperado:**
```
Loading SAM2 model (DEBUG MODE)...
[DEBUG] Checkpoint path: ...
[DEBUG] Model config: configs/sam2.1/sam2.1_hiera_l.yaml
[DEBUG] Device: cuda
[DEBUG] Importing sam2.build_sam...
[DEBUG] Importing sam2.sam2_image_predictor...
[DEBUG] Imports complete
[DEBUG] Calling build_sam2()...
[DEBUG] build_sam2() complete
[DEBUG] Creating SAM2ImagePredictor...
[DEBUG] SAM2ImagePredictor created
```

O último `[DEBUG]` antes do travamento indica onde está o problema.

---

## Diferença CLI vs Chamada Direta

**Por que chamada direta funciona mas CLI não?**

Chamada direta:
```python
from sam2.build_sam import build_sam2
model = build_sam2(...)  # Só isso
```

CLI faz mais coisas ao importar `sam_mosaic`:
1. `warnings.filterwarnings(...)` - Silencia warnings
2. `os.environ["OMP_NUM_THREADS"] = "1"` - **SUSPEITO!**
3. Imports de vários módulos (config, pipeline, api)

---

## Histórico de Testes

### PC1
- [x] CLI funciona normalmente
- [x] Debug mode funciona
- [x] Tempo: ~18 min para 10k×10k
- [x] VRAM estável (~7.6 GB)

### PC2
- [ ] Verificar configuração do PC
- [ ] Verificar checkpoint (MD5)
- [ ] Testar com OMP_NUM_THREADS=4
- [ ] Testar debug mode
- [ ] Reportar último [DEBUG] antes de travar

---

## Arquivos Relevantes

- `sam_mosaic/__init__.py` - Define OMP_NUM_THREADS
- `sam_mosaic/sam/predictor.py` - Carrega modelo SAM2
- `sam_mosaic/core/pipeline.py` - Orquestra o pipeline
- `environment.yml` - Ambiente conda para replicar

---
