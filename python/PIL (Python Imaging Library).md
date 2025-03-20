PIL — это библиотека для работы с изображениями в Python. Она предоставляет инструменты для открытия, обработки и сохранения изображений в различных форматах. Современная версия библиотеки называется **Pillow** (форк оригинального PIL).

* PIL используется для обработки изображений
* Tensor используется для машинного обучения
## Преобразование PIL в tensor

```python
from torchvision.transforms import v2

from PIL import Image

image = Image.open('example.jpg')

# v2.ToImage:
# 1. конвертирует в tensor PyTorch;
# 2. Меняет порядок осей с (h, w, c) (как в PIL) на (c, h, w) (как в PyTorch)
# 3. Нормализует значения пикселей в диапазоне [0, 1]

transform = v2.ToImage()
tensor_image = transform(image)

print(tensor_image.shape) # [3, h, w]
print(tensor_image.dtype) # torch.float32

```