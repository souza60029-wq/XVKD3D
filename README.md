# XVKD3D — VKD3D-Proton + ESX (Samsung Xclipse) fixes

Fork de build do [VKD3D-Proton](https://github.com/HansKristian-Work/vkd3d-proton)
focado em Exynos / Samsung Xclipse (vendorID `0x144d`).

## O que muda vs upstream v3.0.1

| # | Arquivo | Mudança |
|---|---------|---------|
| 1 | `include/vkd3d.h` | Nova flag `VKD3D_CONFIG_FLAG_ESX_XCLIPSE_WORKAROUNDS` (bit 63 — único livre no v3.0.1; bit 59 do patch antigo colidia com `DAMAGE_NOT_ZEROED_ALLOCATIONS`). Aliases `ESX_NO_RT_LIBRARIES` / `ESX_SWAPCHAIN_DEVICE_WAIT` mantidos por compatibilidade |
| 2 | `libs/vkd3d/device.c` | Registra `esx_xclipse_workarounds` (+ aliases) no `VKD3D_CONFIG`; `nodxr` não finge mais `RaytracingTier 1.1` no override de feature level |
| 3 | `libs/vkd3d/raytracing_pipeline.c` | Em Xclipse: nunca cria RT pipeline libraries (`LIBRARY_BIT` segfaulta o driver) — stripa `ALLOW_STATE_OBJECT_ADDITIONS`, deferra `COLLECTION` |
| 4 | `libs/vkd3d/swapchain.c` | Em Xclipse: `vkDeviceWaitIdle` antes de destruir swapchain (só `vkQueueWaitIdle` não libera CMA → OOM ~91% RAM no Exynos) |
| 5 | `libs/vkd3d/resource.c` | Só logging: `WARN` mostra DXGI format / VkFormat / feature flags quando validação falha |

Tudo é gated em `vendorID == 0x144d` **OU** na flag — outras GPUs pagam custo zero.

## Build (CI)

`.github/workflows/build.yml` clona o upstream `v3.0.1`, aplica `patches/esx-xclipse-v4.patch`
com `git apply --check`, compila `x64 + x86` via `package-release.sh` e empacota o `.wcp`.

Artefatos: `XVKD3D-<versão>-3.0.1.wcp` + `sha256.txt`.

## Uso

```ini
# Winlator / GameHub: custom VKD3D config
VKD3D_CONFIG=esx_xclipse_workarounds
```

Sem a flag, os workarounds ativam sozinhos em GPU Samsung. A flag só serve para
forçar o comportamento em outras GPUs (teste) — ou use os aliases antigos
`esx_no_rt_libraries` / `esx_swapchain_device_wait`.

## Versões

- `XVKD3D 0.5` → VKD3D-Proton 3.0.1 + `patches/esx-xclipse-v4.patch`
- `XVKD3D 0.4` → VKD3D-Proton 3.0.1 + `esx3.patch` (obsoleto: bit 59 colide com upstream!)

## Licença

LGPL-2.1, como o upstream. Vide `LICENSE.upstream.md`.
