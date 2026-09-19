## Полезности



```python
# Создание и активация окружения
python3 -m venv myenv # создание
source myenv/bin/activate # активация
deactivate # выход из окружения

# Найти расположение пакета
import torch

print(torch.__file__)

# Создание файла зависимостей
pip freeze > requirements.txt
```