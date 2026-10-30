# Métricas del Paralelismo

Estado: Sin empezar
ítem principal: Teorías PC (Teor%C3%ADas%20PC%202520675b4f6f80a7b401ea2c307583f0.md)

![image.png](M%C3%A9tricas%20del%20Paralelismo/image.png)

![image.png](M%C3%A9tricas%20del%20Paralelismo/image%201.png)

## Speedup

Tp=Tiempo paralelo (Tiempo que se tarda desde que empieza el primer proceso o hilo, hasta que terminan de ejecutarse el último de ellos)

Ts=Tiempo secuencial (Tiempo que tardaría en ejecutarse la mejor solución secuencial al problema) (Ejecutado sobre la mejor máquina)

Si dos aplicaciones corren en distintas máquinas, no me alcanza con comparar los tiempos de ejecución. Por eso tenemos que calcular el **speedup**

![image.png](M%C3%A9tricas%20del%20Paralelismo/image%202.png)

Si el speedup, da menor o igual que 1, significa que la solución paralela es más lenta que la secuencial, por lo tanto no tendría sentido implementarla.

Potencia de calculo de cada una de las máquinas, dividido, Potencia de calculo de la mejor de las máquinas.

Si las arquitectura que estamos usando homogénea, es decir, todas las unidades de procesamiento son iguales, entonces, el SpeedUp óptimo es igual a la cantidad de unidades de procesamiento (P). Entonces la eficiencia, para este caso, es igual a SpeedUp sobre cantUnidadesProcesamiento (S/P)

### Eficiencia

![image.png](M%C3%A9tricas%20del%20Paralelismo/image%203.png)

## Limitantes del Speedup

![image.png](M%C3%A9tricas%20del%20Paralelismo/image%204.png)

---

![image.png](M%C3%A9tricas%20del%20Paralelismo/image%205.png)