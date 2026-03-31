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
