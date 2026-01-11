# Kykar — landing / визитка стримера

Лендинг-визитка для стримера **Kykar (Кук)**: ссылки на соцсети, “Обо мне”, “Контент”, “Сетап”, “Контакты”.

## Медиа (что куда положить)

- **Фото в шапке**: положи файл в `public/` под одним из имён:
  - `public/logo.jpeg` (приоритет)
  - `public/logo.jpg`
  - `public/logo.png`
  - `public/streamer.webp`
  - `public/streamer.png`
  - `public/streamer.jpg`
  Если фото нет — будет показан `public/streamer-placeholder.svg`.
- **Фон-видео** (опционально): `public/arc-raiders-bg.mp4` (и опционально `public/arc-raiders-bg.jpg` как poster)
- **Анимированный логотип** (опционально): `public/arc-raiders-logo.mp4` (и опционально `public/arc-raiders-logo.jpg` как poster)

## Команды

```bash
npm i
npm run dev
npm run build
npm run preview
```
