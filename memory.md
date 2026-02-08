# Memory — Super Ninja Hyper Mega Ultra

## Estado actual
- Versión: v51
- Archivo principal: `proto/superninja-static.html`
- Backup estable: `proto/superninja-stable-1.html` (pre-camera, v36)

## Historial de sesiones

### 2026-01-29 — Sesión inicial
- Creación del juego desde cero
- Mecánicas base: movimiento, melee, dash, shooting, witch time
- Sistema de enemigos: normal, big, boss
- Sistema de power-ups con slot machine
- Primer stable (v36)

### 2026-01-30 — Camera + Gamepad + Polish
- Sistema de cámara Hotline Miami (v37+)
- Fix gamepad: polling de alta frecuencia (~1000Hz) para igualar responsividad de teclado
- Sistema de combo melee (cada 3er hit = Combo Finisher violet)
- Deflexión de láser con charged melee o combo finisher
- Enemy hit effects (bounce/squish)
- Perfect Clear: matar todos en Witch Time → +1 a todos los powerups
- Re-corte de mitades usa detección por distancia (más generoso)
- Limpieza de comentarios históricos
- Llegó a v51

## Notas técnicas
- Todo en un solo HTML, sin dependencias externas
- Canvas 2D, assets procedurales
- Gamepad necesita polling independiente del game loop para capturar taps rápidos
- El sistema de fases de muerte (slashDeath) tiene 7 estados: flash→split→suspended→half→quarter→explode (+piece legacy)

## Próximos pasos
- Pruebas mobile (touch controls)
