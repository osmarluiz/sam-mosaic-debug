# SAM-Mosaic Debug - Memory Leak Investigation

## Objetivo
Investigar por que o SAM-Mosaic apresenta memory leak (VRAM) no PC2 mas nao no PC1, usando a mesma imagem 10k x 10k.

---

## PC1 (Osmar-WSL) - SEM LEAK

**Ambiente:**
```
SAM2: 1.1.0 (PyPI - fork JinsuaFeito-dev)
PyTorch: 2.5.1+cu124
CUDA: 12.4
```

**Teste com imagem 10k x 10k:**
- 100 tiles (10x10)
- Tempo: 17.8 min (codigo otimizado)
- VRAM: Estavel em ~7.6-8.1 GB
- Crescimento: Nenhum (variacao < 200 MB)
- **Resultado: SEM MEMORY LEAK**

---

## PC2 - COM LEAK

**Ambiente:**
```
SAM2: (preencher - pip show sam2)
PyTorch: (preencher)
CUDA: (preencher)
Driver NVIDIA: (preencher)
```

**Teste com imagem 10k x 10k:**
- (rodar o teste e preencher resultados)

**Comandos para diagnostico:**
```bash
# 1. Versoes instaladas
pip show sam2
python -c "import torch; print(f'PyTorch: {torch.__version__}'); print(f'CUDA: {torch.version.cuda}')"
nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv

# 2. Rodar o teste com a imagem 10k x 10k
sam-mosaic /caminho/para/image_10k.tif /output/ --checkpoint checkpoints/sam2.1_hiera_large.pt

# 3. Monitorar VRAM durante o teste (em outro terminal)
watch -n 5 "nvidia-smi --query-gpu=memory.used --format=csv,noheader"
```

---

## Hipoteses

1. **Versao SAM2 diferente**: PC1 usa versao PyPI (1.1.0), PC2 pode usar versao oficial Facebook
2. **Versao PyTorch/CUDA diferente**: Memory management varia entre versoes
3. **Driver NVIDIA diferente**: Drivers mais antigos podem ter bugs de memoria

---

## Acao necessaria no PC2

1. Rodar os comandos de diagnostico acima
2. Testar com a mesma imagem 10k x 10k
3. Reportar VRAM inicial e final
4. Atualizar este arquivo com os resultados

---

## Historico de comunicacao

### [PC1 -> PC2] Mensagem inicial
Estamos investigando um memory leak que ocorre no PC2 mas nao no PC1.
Precisamos que voce:
1. Execute os comandos de diagnostico listados acima
2. Rode o sam-mosaic na imagem 10k x 10k
3. Monitore a VRAM durante o processamento
4. Atualize este arquivo com os resultados

A imagem de teste esta disponivel em: (adicionar link ou caminho)

---
