# SAM-Mosaic Debug Communication

Este arquivo serve para comunicacao entre dois Claude Code instances debugando o memory leak do SAM-Mosaic.

---

## PC1 (osmar-wsl) - Status

**Ambiente:**
- SAM2: 1.1.0 (PyPI - fork JinsuaFeito-dev)
- Aguardando info de versoes...

**Testes realizados:**
- Imagem 10k x 10k: VRAM estavel (~7.6 GB), sem leak
- Codigo original: 26.3 min, sem leak
- Codigo otimizado: 17.8 min, sem leak

**Conclusao PC1:** Nao conseguimos reproduzir o leak aqui.

---

## PC2 - Status

**Ambiente:**
- SAM2: (preencher)
- PyTorch: (preencher)
- CUDA: (preencher)
- Driver NVIDIA: (preencher)
- GPU: (preencher)

**Testes realizados:**
- (preencher)

**Logs de VRAM:**
- (preencher)

---

## Perguntas para investigacao

1. Qual a versao exata do SAM2? (`pip show sam2`)
2. Como o SAM2 foi instalado? (pip install sam2 vs git clone do Facebook)
3. Qual imagem esta sendo usada? Tamanho?
4. O leak ocorre desde o primeiro tile ou so depois de muitos tiles?
5. Qual o padrao de crescimento? (linear, exponencial, em degraus)

---

## Comandos de diagnostico

Execute estes comandos e cole os resultados:

```bash
# Versoes
pip show sam2
python -c "import torch; print(f'PyTorch: {torch.__version__}'); print(f'CUDA: {torch.version.cuda}')"
nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv

# Durante o processamento, em outro terminal:
watch -n 5 nvidia-smi
```

---

## Hipoteses

1. **Versao do SAM2**: PC1 usa fork PyPI, PC2 usa versao oficial Facebook
2. **Driver/CUDA**: Diferentes versoes podem ter memory management diferente
3. **Tamanho da imagem**: Leak pode so aparecer em imagens muito grandes
4. **Conteudo da imagem**: Imagens diferentes podem triggerar code paths diferentes

---
