
```python
import matplotlipb.pyplot as plt

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))

ax1.xlabel('Номер итерации')
ax2.ylabel('Значение x')
ax1.legend(['a', 'b', 'c'], title="С методом импульсов")  
ax1.grid(True)  
ax1.plot([1,2,3,4,5,6], [1, 3,4,5,6, 1])

plt.show()
```