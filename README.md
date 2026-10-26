# OpenWrt package CI

Shared GitHub Actions infrastructure for building OpenWrt packages with the OpenWrt SDK.
It contains the common build helper, target matrix generator, and reusable workflow.

## Описание

Общая инфраструктура GitHub Actions для сборки пакетов OpenWrt с помощью OpenWrt SDK.
Содержит скрипт сборки, генератор матрицы целевых платформ и переиспользуемый workflow.

Общая логика сборки вынесена сюда, чтобы не хранить одинаковые копии `openwrt-build.sh`, `openwrt-matrix.py` и GitHub Actions workflow в каждом репозитории пакета.

Репозитории пакетов содержат только собственные файлы пакета, `openwrt-build.env` и небольшой workflow, который вызывает reusable workflow из этого репозитория.

## Структура проекта

- `.github/workflows/openwrt-build.yml` - общий reusable GitHub Actions workflow;
- `.github/workflows/check.yml` - проверки самого CI-репозитория;
- `openwrt-build.sh` - сборка пакетов через OpenWrt SDK;
- `openwrt-matrix.py` - генератор списка OpenWrt target/subtarget для matrix build.

## Использование

В репозитории OpenWrt-пакета находится небольшой workflow:

```yaml
jobs:
  build:
    uses: karen07/openwrt-package-ci/.github/workflows/openwrt-build.yml@main
```

При запуске CI reusable workflow получает содержимое вызывающего репозитория, загружает общие build scripts из `openwrt-package-ci` и запускает сборку с параметрами из `openwrt-build.env`.

Пример `openwrt-build.env`:

```sh
PACKAGE_DIRS="
antiblock
"

PACKAGE_OUTPUTS="
"
```

Если имя создаваемого APK отличается от имени каталога пакета, это можно указать через `PACKAGE_OUTPUTS`:

```sh
PACKAGE_OUTPUTS="
amneziawg=kmod-amneziawg
"
```

## Сборка

Конкретный список OpenWrt-версий, target и subtarget формируется `openwrt-matrix.py`.

Сам пакет собирается `openwrt-build.sh` в соответствующем OpenWrt SDK.

Изменения, добавленные в ветку `main` этого репозитория, будут использоваться последующими CI-запусками всех package-репозиториев, которые ссылаются на:

```yaml
uses: karen07/openwrt-package-ci/.github/workflows/openwrt-build.yml@main
```

Таким образом, общую логику CI достаточно изменить в одном месте.
