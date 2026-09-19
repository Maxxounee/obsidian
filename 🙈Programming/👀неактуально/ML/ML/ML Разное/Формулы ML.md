### Средний эмпирический риск (функционал качества)


$$
\begin{align}
& Q(a, X^l)=\frac{1}{l}\sum^{l}_{i=1}L(a, x_{i})\\
\\ \\
&L \text{ — Функция потерь}\\
&a\text{ — Алгоритм/модель с параметрами. В линейной регрессии это фактический набор весов:} \\
&a = (w, b),\text{ то есть модель } y=wx+b \\ \\ \\
&\text{Средний эмпирический риск при квадратической функции потерь:} \\ \\

&Q(a, X) = \frac{1}{n}(X \cdot w - Y)^{2}=0  \\ \\
&\text{Дифференцирование выражения по вектору } w \text{:} \\
 \\

&\frac{dQ(a, X)}{dw}=\frac{2}{n}\cdot X^{T}\cdot (X\cdot w - Y) = 0 \\
 \\ \\  
&\dots \\ \\\\
&\text{Частная производная по вектору параметров }w\text{ в векторно-матричном виде:} \\
&\text{(Пример для модели } a(x) = w_{0}+w_{1}\cdot x+w_{2}\cdot x^{2} + w_{3}\cdot x^{3}\text{)} \\ \\
&Q(w) = \frac{1}{n} \cdot \sum^{n}_{i=1}(a(x_{i})-f(x_{i}))^{2}\qquad  

\frac{dQ(w)}{dw}=\frac{2}{n}\sum^{n}_{i=1}(w^{T} \cdot s_{i}-f(x_{i}))\cdot s_{i}^{T} \\
\\
&s_{i} = [1, x, x^{2}, x^{3}]^{T}\text{ — вектор признаков i-го образа обучающей выборки}
\end{align}
$$


### Градиентный спуск (наискорейший спуск)

$$
\begin{align}
&x_{n+1}=x_{n} - \lambda_{n} \cdot \frac{df(x)}{dx} \\ 
&x_{n+1}=x_{n} - \lambda_{n} \cdot sign(\frac{df(x)}{dx})\\ \\
&\lambda \text{ — шаг сходимости}  \\ \\
&\dots \\ \\ 
&\text{Стохастический градиентный спуск (SGD):}  \\ \\
&w^{(t+1)}=w^{(t)} - \eta_{t} \cdot \bigtriangledown L_{k}(w^{(t)}, x_{k}) \\ \\ 
&\bigtriangledown \bar{Q}(w^{(t)})= \bigtriangledown L_{k}(w^{(t)}, x_{k})  
\text{ — псевдоградиент; k — случайный индекс} \\
&\text{ вектора из обучающей выборки}
\end{align}

$$

```python
Qe = 0
lm = 0.01
eta = np.array([0.1, 0.01, 0.001, 0.0001])

def loss(w, x, y):
	return (w @ x - y) ** 2
	
def df(w, x, y):
    return 2 * ((w.T @ x.T - y) @ x)

# SAG

for i in range(N):  
    eps_k = np.average([loss(w, x, y) for x, y in zip(x_train, y_train)])
    w = w - eta * sum([df(w, x, fx)] for x, y in zip(x_train, y_train)) / len(x_train)

Q = np.average((w @ x_train - y_train) ** 2)
    
# SGD 

for i in range(N):  
	k = rand_k()  
	
    x_k = x_train[k]  
    y_k = y_train[k]  
  
    eps_k = loss(w, x_k, y_k)  
    w = w - eta * df(w, x_k, y_k)  
  
    Qe = lm * eps_k + (1 - lm) * Qe

# SGD батчами

for i in range(N):  
	k = rand_k()  
	
    x_k = x_train[k:k+10]  
    y_k = y_train[k:k+10]  
  
    eps_k = np.average([loss(w, x, y) for x, y in zip(x_k, y_k)])
    w = w - eta * [df(w, x, y) for x, y in zip(x_k, y_k)].mean(1)  
  
    Qe = lm * eps_k + (1 - lm) * Qe
```
### Поиск коэффициентов

$$
w^{T}=\sum_{i=1}^{l}x^{T}_{i}\cdot y_{i} \cdot \left( \sum_{i=1}^{l}x_{i} \cdot x^{T}_{i} \right)^{-1} 
$$

### МНК Метод наименьших квадратов

$$
\begin{align}
&\omega_{*}=(X^{T}\cdot X+\lambda \cdot I)^{-1} \cdot X^{T} \cdot Y \\ 
\\
&I_{n\times n} \text{ — единичная матрица}
\end{align}
$$

Вариант 1 (оригинальный. С L2 регуляризацией):

```python
import numpy as np
import matplotlib.pyplot as plt


x = np.arange(0, 10.1, 0.1)
y = np.array([a ** 3 - 10 * a ** 2 + 3 * a + 500 for a in x])  # функция в виде полинома x^3 - 10x^2 + 3x + 500
x_train, y_train = x[::2], y[::2]
N = 13  # размер признакового пространства (степень полинома N-1)
L = 20  # при увеличении N увеличивается L (кратно): 12; 0.2   13; 20    15; 5000

X = np.array([[a ** n for n in range(N)] for a in x])  # матрица входных векторов
IL = np.array([[L if i == j else 0 for j in range(N)] for i in range(N)])  # матрица lambda*I
IL[0][0] = 0  # первый коэффициент не регуляризуем
X_train = X[::2]  # обучающая выборка
Y = y_train  # обучающая выборка

# вычисление коэффициентов по формуле w = (XT*X + lambda*I)^-1 * XT * Y
A = np.linalg.inv(X_train.T @ X_train + IL)
w = A @ X_train.T @ Y
print(w)

```

Вариант 2:

```python
import numpy as np  
  
  
def func(x):  
    return 0.5 * x + 0.2 * x ** 2 - 0.05 * x ** 3 + 0.2 * np.sin(4 * x) - 2.5  
  
  
def model(w, x):  
    return w[0] + w[1] * x + w[2] * x ** 2 + w[3] * x ** 3  
  
  
coord_x = np.arange(-4.0, 6.0, 0.1)  
  
x_train = np.array([[_x**i for i in range(4)] for _x in coord_x]) # обучающая выборка  
y_train = func(coord_x) # целевые выходные значения  
  
# здесь продолжайте программу  
  
X = x_train  
w = np.linalg.inv(X.T @ X) @ X.T @ y_train  
  
Q = np.mean([(func(_x) - model(w, _x)) ** 2 for _x in coord_x])  
# print(w, Q)
```