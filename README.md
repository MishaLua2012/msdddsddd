# MSDF Atlas Generator (GitHub Actions + Termux)

Готовый набор для генерации MSDF-атласов (`.png` + `.json`) без компьютера — вся сборка идёт на GitHub, с телефона только пуш и запуск workflow.

## Что внутри

```
.
├── .github/workflows/msdf.yml   # сам workflow
├── fonts/                       # сюда клади свой .ttf/.otf
├── charset.txt                  # набор символов (кириллица+латиница+пунктуация по умолчанию)
├── atlas/                       # сюда упадёт результат (.png + .json)
└── README.md
```

## Шаг 1. Загрузи проект в свой репозиторий

Распакуй zip и залей содержимое в корень своего GitHub-репозитория (или создай новый репозиторий из этих файлов).

## Шаг 2. Положи шрифт

Скопируй свой `.ttf` или `.otf` файл в папку `fonts/`, например:

```
fonts/MyFont.ttf
```

## Шаг 3. (опционально) Настрой набор символов

Файл `charset.txt` уже содержит кириллицу, латиницу, цифры и базовую пунктуацию.
Если нужны другие символы — просто отредактируй этот файл (обычный текст, симовлы через пробел/подряд).

## Шаг 4. Установка Termux и запуск

```bash
pkg update && pkg upgrade -y
pkg install git gh -y

# логин в GitHub (один раз)
gh auth login

# клонируем свой репозиторий
git clone https://github.com/USERNAME/REPO.git
cd REPO

# кладём шрифт, если ещё не сделано
cp /sdcard/Download/MyFont.ttf fonts/

git add .
git commit -m "add font"
git push
```

## Шаг 5. Запуск генерации атласа

```bash
gh workflow run msdf.yml \
  -f font_file=fonts/MyFont.ttf \
  -f font_size=32 \
  -f atlas_name=myatlas
```

Проверка статуса:

```bash
gh run list --workflow=msdf.yml
gh run watch
```

## Шаг 6. Скачать результат

Способ А — атлас автоматически закоммитится в папку `atlas/` в репозитории, можно просто:

```bash
git pull
ls atlas/
```

Способ Б — скачать как GitHub Actions Artifact:

```bash
gh run list --workflow=msdf.yml --limit 1
gh run download <RUN_ID> -n msdf-atlas -D ./downloaded
```

## Настройки, которые можно менять при запуске

| Параметр       | Описание                                  | По умолчанию      |
|----------------|--------------------------------------------|-------------------|
| `font_file`    | путь к шрифту в репо                       | `fonts/MyFont.ttf`|
| `font_size`    | размер шрифта в px                         | `32`              |
| `atlas_name`   | имя выходных файлов (без расширения)       | `atlas`           |
| `msdf_version` | тег релиза msdf-atlas-gen на GitHub        | `v1.3`            |

Пример с другим именем и размером:

```bash
gh workflow run msdf.yml -f font_file=fonts/Roboto.ttf -f font_size=48 -f atlas_name=roboto_48
```

## Автозапуск при пуше

Workflow также запускается автоматически при любом пуше, затрагивающем `fonts/`, `charset.txt` или сам workflow-файл — то есть достаточно просто закинуть новый шрифт и запушить, вручную `gh workflow run` не обязателен.

## Если что-то пошло не так

- **"файл шрифта не найден"** — проверь, что путь в `font_file` совпадает с реальным путём в репозитории (регистр важен).
- **Ошибка скачивания бинарника** — попробуй указать другую версию через `-f msdf_version=vX.Y` (актуальные версии смотри на странице релизов: https://github.com/Chlumsky/msdf-atlas-gen/releases). Workflow также пытается автоматически подхватить последний релиз, если указанная версия не найдена.
- **Нет прав на `gh workflow run`** — убедись, что сделал `gh auth login` и что у токена есть права `repo` + `workflow`.
