# Opciones-sobre-Acciones-Argentinas
Proyecto con base en el modulo 6 de Bases de la Inversion y Financiacion# Módulo 6 — Estrategias de Opciones sobre ADRs Argentinos

## Descripción

Análisis de opciones sobre el ADR de YPF (NYSE: YPF) con tres ejes:

1. **Verificación de paridad put-call** — contraste entre el modelo europeo y los límites de arbitraje para opciones americanas, con análisis de eficiencia de mercado neto de costos de transacción.
2. **Construcción de estrategias** — implementación de Protective Put, Covered Call y Collar sobre un mismo subyacente y vencimiento.
3. **Análisis de payoff** — diagramas de P&L al vencimiento con pérdida/ganancia máxima y break-even analíticos.

Este proyecto forma parte de una serie de módulos de portafolio con enfoque en finanzas de mercado (inversiones y gestión de riesgo).


## Datos utilizados

| Parámetro | Valor |
| Subyacente | YPF S.A. ADR (NYSE: YPF) |
| Vencimiento analizado | 2026-11-20 |
| Fecha de extracción | 2026-08-10 |
| Precio spot (S₀) | extraído vía `yfinance` |
| Tasa libre de riesgo | 5.25% anual (T-Bill 3M) |
| Tiempo al vencimiento (T) | 102 días / 365 |

**Fuente:** Yahoo Finance — cadena de opciones via `yfinance`. Se filtraron strikes con `openInterest ≤ 50` o `bid = 0` por ausencia de liquidez real.


## Metodología

### Fase 1 — Extracción y limpieza

Se descargó la cadena de opciones completa para el vencimiento seleccionado. El precio mid `(bid + ask) / 2` se usó como proxy del precio de mercado para evitar el sesgo del spread.

### Fase 2 — Verificación de paridad put-call

La paridad put-call exacta aplica a **opciones europeas**:

$$C - P = S - K \cdot e^{-rT}$$

Las opciones sobre ADRs en NYSE son de ejercicio **americano**. Para opciones americanas sobre acciones sin dividendos, la paridad se convierte en desigualdad:

$$S - K \leq C_A - P_A \leq S - K \cdot e^{-rT}$$

La diferencia entre el lado izquierdo de mercado y el valor teórico europeo representa la **prima de ejercicio anticipado** incorporada en las puts americanas. Esto no es una violación de no-arbitraje sino una consecuencia de la opcionalidad adicional.

El arbitraje neto se calcula como:

$$\text{Arbitraje neto} = |\text{Desvío europeo}| - \text{Costo de transacción}$$

donde el costo de transacción se aproxima como el promedio de los bid-ask spreads de call y put. Un valor negativo implica que el mercado es eficiente dentro de los costos observables.

### Fase 3 — Estrategias

Los strikes se seleccionaron dinámicamente: K_p es el strike inmediatamente por debajo del ATM y K_c el inmediatamente por encima.

| Estrategia | Posición | Objetivo |
| **Protective Put** | Long stock + Long put (K_p) | Seguro contra caída; upside ilimitado |
| **Covered Call** | Long stock + Short call (K_c) | Ingreso por prima; upside acotado |
| **Collar** | Long stock + Long put (K_p) + Short call (K_c) | Pérdida y ganancia acotadas; puede ser de costo neto reducido |

Las métricas de cada estrategia (pérdida máxima, ganancia máxima, break-even) se calculan analíticamente, no mediante aproximación numérica sobre el rango de simulación.

#### Fórmulas analíticas

| Métrica | Protective Put | Covered Call | Collar |
| Pérdida máxima | K_p − S₀ − p_p | p_c − S₀ | K_p − S₀ − p_p + p_c |
| Ganancia máxima | Ilimitada | K_c − S₀ + p_c | K_c − S₀ − p_p + p_c |
| Break-even | S₀ + p_p | S₀ − p_c | S₀ + p_p − p_c |


## Notas

- El análisis de pricing teórico (Black-Scholes, volatilidad implícita, Greeks) corresponde a un módulo posterior y no está incluido en este proyecto.
- La fecha de extracción está hardcodeada en el notebook (`datetime(2026, 8, 10)`). Para replicar con datos en tiempo real, reemplazar por `datetime.today()`.
- YPF puede distribuir dividendos esporádicamente. Si hay un ex-dividend dentro del período analizado, la paridad put-call requiere ajuste: $C - P = S - \text{PV}(D) - K e^{-rT}$. Verificar con `ypf.dividends` antes de ejecutar.

