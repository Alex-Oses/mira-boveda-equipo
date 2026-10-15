# Decisiones

Una entrada por decisión, las más nuevas arriba. Formato:

```
### AAAA-MM-DD · Título
- Decisión:
- Por qué:
- Quién:
```

### 2026-10-14 · Android con Expo, iOS como PWA
- **Decisión:** Android usa la app Expo de `mobile/` (Expo Push, que por debajo usa FCM). iOS, de momento, usa la web como PWA con Web Push (iOS 16.4+, instalada en la pantalla de inicio) hasta que haya cuenta de Apple Developer.
- **Por qué:** la app Expo ya existe y la cuenta de Apple cuesta 99 €/año.
- **Quién:** Alex (propuesta; falta que el equipo la confirme).

### 2026-10-14 · Un solo endpoint de tokens para los dos tipos
- **Decisión:** `POST /v1/push/tokens` con `{ token, tipo: "expo" | "web", plataforma, idioma }`, guardado en `usuarios/{uid}/dispositivos/{id}`. `DELETE /v1/push/tokens/:id` al cerrar sesión. `enviarPush(uid, { titulo, cuerpo, data })` envía por el canal de cada token.
- **Por qué:** la app Android y la PWA comparten el mismo backend.
- **Quién:** Alex (propuesta).
