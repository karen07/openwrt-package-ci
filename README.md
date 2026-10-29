# OpenWrt Package CI

Shared GitHub Actions infrastructure for building OpenWrt packages with the OpenWrt SDK.
It contains the common build helper, target matrix generator, and reusable workflow.

Общая инфраструктура GitHub Actions для сборки пакетов OpenWrt с помощью OpenWrt SDK.
Содержит скрипт сборки, генератор матрицы целевых платформ и переиспользуемый workflow.

## Описание

Общая логика сборки вынесена сюда, чтобы не хранить одинаковые копии `openwrt-build.sh`, `openwrt-matrix.py` и GitHub Actions workflow в каждом репозитории пакета.

Репозитории пакетов содержат только собственные файлы пакета, `openwrt-build.env` и небольшой workflow, который вызывает reusable workflow из этого репозитория.

## Разработка и проверка изменений

**Основные точки изменения:**

- `openwrt-build.sh` - общий сценарий сборки пакетов с помощью OpenWrt SDK;
- `openwrt-matrix.py` - поиск и фильтрация target/subtarget для матрицы;
- `.github/workflows/openwrt-build.yml` - переиспользуемый workflow GitHub Actions.

**Поток сборки:** `workflow_call` получает `tag_name`, `package`, `include`
и `exclude`. Задача `generate-config` удаляет ведущую `v` из версии и
вызывает `openwrt-matrix.py`, который читает каталог релизов OpenWrt,
фильтрует `target/subtarget` и получает `pkgarch`. Задача `build` создаёт
job для каждой записи матрицы, копирует `openwrt-build.sh` в каталог
вызывающего проекта и собирает пакет с помощью SDK. Результат публикуется
как `release/*.apk` (или как artifact при запуске через `act`).

**Граница конфигурации:** `openwrt-build.sh` загружает `openwrt-build.env`
из каталога, где расположен сам скрипт. Именно поэтому workflow копирует
его в репозиторий пакета. `PACKAGE_DIRS` задаёт каталоги исходных пакетов,
а `PACKAGE_OUTPUTS` сопоставляет их с именами выходных APK. Изменения
параметров вызова, переменных окружения и схемы JSON-матрицы нужно
согласовывать между тремя файлами и вызывающими репозиториями.

**Проверка изменений:** При правке матрицы проверьте нормализацию версии,
include/exclude, пустую выборку и получение `pkgarch`; при правке сборки -
режимы `install`, `build-router`, `build-target` и параметры
`openwrt-build.env`. Изменения reusable workflow затрагивают все
репозитории `*-openwrt-package`, особенно использующие `@main`.
Проверка выполнения CI требует GitHub Actions и доступа к SDK.

## Сборка

Конкретный список OpenWrt-версий, target и subtarget формируется `openwrt-matrix.py`.

Сам пакет собирается `openwrt-build.sh` в соответствующем OpenWrt SDK.

Изменения, добавленные в ветку `main` этого репозитория, будут использоваться последующими CI-запусками всех package-репозиториев, которые ссылаются на:

```yaml
uses: karen07/openwrt-package-ci/.github/workflows/openwrt-build.yml@main
```

Таким образом, общую логику CI достаточно изменить в одном месте.

**Граница совместимости:** ссылки вида `@main` не закрепляют версию общего
workflow. Новые изменения в `main` могут повлиять на последующие сборки
вызывающих репозиториев без правок их собственного кода. При изменении
входных параметров workflow или формата `openwrt-build.env` следует
одновременно сверять все репозитории-потребители.

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

## Структура проекта

- `.github/workflows/openwrt-build.yml` - общий reusable GitHub Actions workflow;
- `openwrt-build.sh` - сборка пакетов через OpenWrt SDK;
- `openwrt-matrix.py` - генератор списка OpenWrt target/subtarget для matrix build.

## Лицензия

Проект распространяется под лицензией GNU Affero General Public License v3.0. Подробности см. в файле [LICENSE](LICENSE).
