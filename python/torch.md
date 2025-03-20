- [[#Настройка устройства для вычисления|Настройка устройства для вычисления]]
- [[#Создание датасетов|Создание датасетов]]
	- [[#Создание датасетов#Mnist|Mnist]]
	- [[#Создание датасетов#Изображения квадратов|Изображения квадратов]]
- [[#Обработка датасетов|Обработка датасетов]]
- [[#Вариант обработки датасетов с помощью класса ImageFolder (torchvision)|Вариант обработки датасетов с помощью класса ImageFolder (torchvision)]]
- [[#Данные для задачи регрессии (изображения с белыми квадратами)|Данные для задачи регрессии (изображения с белыми квадратами)]]
- [[#Трансформации|Трансформации]]
	- [[#Трансформации#torchvision.transforms|torchvision.transforms]]
	- [[#Трансформации#torchvision.v2|torchvision.v2]]


## Настройка устройства для вычисления

```python
import torch

# Перемещение вычислений на графический процессор (не apple sillicon)
torch.tensor([2, 3], [4, 5], device=torch.device('cuda:0')) 
# apple
tensor = torch.tensor([2, 3], device=torch.device('mps'))

# или

tensor.to('mps');

print(tensor.device) # mps:0

# - `mps` указывает на то, что устройство — это MPS (GPU на Apple Silicon).
    
# :0 — это идентификатор устройства. В данном случае `0` означает, что используется первое (и обычно единственное) доступное устройство MPS.

# **MPS** (Metal Performance Shaders) — это фреймворк от Apple, который позволяет использовать GPU на устройствах с чипами Apple Silicon (M1, M2, M3 и т.д.) для ускорения вычислений.

# ===
# При работе с тензорами, нужно убедиться, что все они лежат на одной железке

tensor_1.to('cpu')
tensor_2.to('mps')

tensor_1 + tensor_2 # Ошибка

```

## Создание датасетов

### Mnist
```python
import struct  
import sys  
import os  
  
from array import array  
from os import path  
from PIL import Image  
  
import numpy as np  
import torchvision.datasets  
  
train_dataset = torchvision.datasets.MNIST(root='./sample_data/', train=True, download=True)  
test_dataset = torchvision.datasets.MNIST(root='./sample_data/', train=False, download=True)  
  
  
def read(dataset):  
    path_root = './sample_data/MNIST/raw/'  
  
    if dataset is 'training':  
        path_img = path_root + 'train-images-idx3-ubyte'  
        path_lbl = path_root + 'train-labels-idx1-ubyte'  
    elif dataset is 'testing':  
        path_img = path_root + 't10k-images-idx3-ubyte'  
        path_lbl = path_root + 't10k-labels-idx1-ubyte'  
    else:  
        raise ValueError('dataset must be testing or training')  
  
    with open(path_lbl, 'rb') as f_label:  
        _, size = struct.unpack('>II', f_label.read(8))  
        lbl = array('b', f_label.read())  
  
    with open(path_img, 'rb') as f_img:  
        _, size, rows, cols = struct.unpack('>IIII', f_img.read(16))  
        img = array('B', f_img.read())  
  
    return lbl, img, size, rows, cols  
  
  
def write_dataset(labels, data, size, rows, cols, output_dir):  
    classes = {i: f"class_{i}" for i in range(10)}  
  
    output_dirs = [  
        path.join(output_dir, classes[i])  
        for i in range(10)  
    ]  
  
    for dirr in output_dirs:  
        if not path.exists(dirr):  
            os.makedirs(dirr)  
  
    for (i, label) in enumerate(labels):  
        output_filename = path.join(output_dirs[label], str(i) + '.jpg')  
        print('writing ' + output_filename)  
  
        with open(output_filename, 'wb') as h:  
            data_i = [  
                data[(i * rows * cols + j * cols): (i * rows * cols + (j + 1) * cols)]  
                for j in range(rows)  
            ]  
            data_array = np.asarray(data_i)  
  
            im = Image.fromarray(data_array)  
            im.save(output_filename)  
  
  
output_path = "./mnist"  
  
for dataset in ['training', 'testing']:  
    write_dataset(*read(dataset), path.join(output_path, dataset))
```
### Изображения квадратов 

```python
import numpy as np  
from PIL import Image  
import json  
import os  
  
img = np.random.randint(0, 50, [100000, 64, 64], dtype=np.uint8)  
  
square = np.random.randint(100, 200, [100000, 15, 15], dtype=np.uint8)  
  
coords = np.empty([100000, 2])  
  
data = {}  
  
for i in range(img.shape[0]):  
  
    x = np.random.randint(20, 44)  
    y = np.random.randint(20, 44)  
  
    img[i, (y - 7):(y + 8), (x - 7):(x + 8)] = square[i]  
  
    coords[i] = [y, x]  
  
    name_img = f'img_{i}.jpeg'  
  
    path_img = os.path.join('./rects/', name_img)  
  
    image = Image.fromarray(img[i])  
    image.save(path_img)  
  
    data[name_img] = [y, x]  
  
with open('./rects/coords.json', 'w') as f:  
    json.dump(data, f, indent=2)
```

## Обработка датасетов
```python
import torch  
import torchvision  
  
from torchvision.datasets import ImageFolder  
from torch.utils.data import Dataset, DataLoader, random_split  
  
import os  
import json  
import numpy as np  
import matplotlib.pyplot as plt  
  
from PIL import Image  
  
  
class MNISTDataset(Dataset):  
    def __init__(self, path, transform=None):  
        # Создание дата сета на основе структуры файлов  
  
        self.path = path  
        self.transform = transform  
  
        self.len_dataset = 0  
        self.data_list = []  
  
        for path_dir, dir_list, file_list in os.walk(path):  
            if path_dir == path:  
                self.classes = sorted(dir_list)  
  
                self.class_to_idx = {  
                    cls_name: i for i, cls_name in enumerate(self.classes)  
                }  
  
                continue  
  
            cls = path_dir.split('/')[-1]  
  
            for name_file in file_list:  
                file_path = os.path.join(path_dir, name_file)  
  
                self.data_list.append((file_path, self.class_to_idx[cls]))  
  
            self.len_dataset += len(file_list)  
  
    def __len__(self):  
        return self.len_dataset  
  
    def __getitem__(self, index):  
        file_path, target = self.data_list[index]  
        sample = np.array(Image.open(file_path))  
  
        if self.transform is not None:  
            sample = self.transform(sample)  
  
        return sample, target  
  
  
train_data_path = './dataset/mnist/training'  
testing_data_path = './dataset/mnist/testing'  
  
train_data = MNISTDataset(train_data_path, None)  
testing_data = MNISTDataset(testing_data_path, None)  
  
for cls, one_hot_position in train_data.class_to_idx.items():  
    # Вывод one hot vector каждого класса ([0, 0, 1, 0, 0, 0, 0, 0, 0 , 0])  
    one_hot_vector = [(i == one_hot_position) * 1 for i in range(10)]  
    print(f"{cls} -> {one_hot_vector}")  
  
img, one_hot_position = train_data[0]  
  
cls = train_data.classes[one_hot_position]  
plt.imshow(img, cmap='gray')  
# plt.show()  
  
# Разделение тренировочных данных на тренировочные и валидационные  
train_data, val_data = random_split(train_data, [0.8, 0.2])  
  
# Создание butch  
train_loader = DataLoader(train_data, batch_size=16, shuffle=True)  
val_loader = DataLoader(val_data, batch_size=16, shuffle=False)  
test_loader = DataLoader(testing_data, batch_size=16, shuffle=False)  
  
print(len(train_data), len(val_data))
```

## Вариант обработки датасетов с помощью класса ImageFolder (torchvision)

```python
from torchvision.datasets import ImageFolder  
from torch.utils.data import DataLoader, random_split  

  
train_data_path = './dataset/mnist/training'  
testing_data_path = './dataset/mnist/testing'

train_data = ImageFolder(root=train_data_path)  
testing_data = ImageFolder(root=testing_data_path)  
  
train_data, val_data = random_split(train_data, [0.8, 0.2])  
  
train_loader = DataLoader(train_data, batch_size=16, shuffle=True)  
val_loader = DataLoader(val_data, batch_size=16, shuffle=False)  
test_loader = DataLoader(testing_data, batch_size=16, shuffle=False)
```


## Данные для задачи регрессии (изображения с белыми квадратами)

```python
import matplotlib.pyplot as plt  
from torchvision.datasets import ImageFolder  
from torch.utils.data import Dataset, DataLoader, random_split  
import os  
import json  
from PIL import Image  
import numpy as np

# Данные для задачи регрессии (квадратики)  
  
class DatasetReg(Dataset):  
    def __init__(self, path, transform=None):  
        self.path = path  
        self.transform = transform  
  
        self.list_name_file = os.listdir(path)  
  
        if 'coords.json' in self.list_name_file:  
            self.list_name_file.remove('coords.json')  
  
        self.len_dataset = len(self.list_name_file)  
  
        with open(os.path.join(self.path, 'coords.json'), 'r') as f:  
            self.dict_coords = json.load(f)  
  
    def __len__(self):  
        return self.len_dataset  
  
    def __getitem__(self, index):  
        name_file = self.list_name_file[index]  
        path_img = os.path.join(self.path, name_file)  
  
        img = np.array(Image.open(path_img))  
        coord = np.array(self.dict_coords[name_file])  
  
        if self.transform is not None:  
            img = self.transform(img)  
  
        return img, coord  
  
  
dataset_rect_path = './dataset/rects'  
  
dataset = DatasetReg(dataset_rect_path)  
  
print(len(dataset))  
print(dataset[10])  
  
img, coord = dataset[94543]  
  
print(f"координаты вектора{coord}")  
  
# Показать центр белого квадрата красной точкой  
# plt.scatter(coord[1], coord[0], marker='o', color='red')  
# plt.imshow(img, cmap='gray')  
# plt.show()  
  
# Разобъем данные на тренировочные, валидационные и тестовые
train_set, val_set, test_set = random_split(dataset, [0.7, 0.1, 0.2])  
  
train_loader = DataLoader(train_set, batch_size=64, shuffle=True)  
val_loader = DataLoader(val_set, batch_size=64, shuffle=False)  
testing_loader = DataLoader(test_set, batch_size=64, shuffle=False)
```

## Трансформации

### torchvision.transforms

```python
from torchvision import transforms  
  
  
import numpy as np  
  
  
from PIL import Image  
  
img = np.array(Image.open('./assets/img/dog.jpg'))  
  
print(type(img))  
print(img.shape)  # (высота, ширина, кол-во цветовых каналов)  
print(img.dtype)  # uint8  
print(f"min {img.min()} max {img.max()}")  # 0 255  
  
# Трансформация 1. Приведение к тензору  
transform = transforms.ToTensor()  
  
img_toTensor = transform(img)  
  
# При переводе в тензор, shape меняется. Теперь на нулевой оси находятся каналы. При работ  
# с нейронками так и должно быть  
print(img_toTensor.shape)  # (каналы, высота, ширина)  
  
# Трансформация 2. Нормализация  
  
# mean - среднее значение. std - стандартное отклонение  
# в данную трансформацию можно передавать только тензоры и только с типом float  
transform = transforms.Normalize(mean=(0.5, 0.5, 0.5), std=(0.5, 0.5, 0.5))  
img_norm = transform(img_toTensor)  
  
print(img_norm.min(), img_norm.max())  # -1.0, 1.0  
  
# Compose. Применение нескольких трансформаций  
  
transform = transforms.Compose(  
    [  
        transforms.ToTensor(),  
        transforms.Normalize(mean=(0.5, 0.5, 0.5), std=(0.5, 0.5, 0.5))  
    ]  
)  
img = transform(Image.open('./assets/img/dog.jpg'))
```

### torchvision.transforms.v2

```python
from torchvision import transforms  
from torchvision.transforms import v2  
  
import numpy as np  
import torch  
  
from PIL import Image  
  
img = np.array(Image.open('./assets/img/dog.jpg'))  
  
# Трансформация 1. Приведение к тензору  
transform = v2.ToTensor()  
  
img_v2 = transform(img)  
  
# Трансформация 2. To Image. Приведение к tv_tensor (То же самое, что и tensor)  
  
transform = v2.ToImage()  
  
img_v2 = transform(Image.open('./assets/img/dog.jpg'))  
  
# Трансформация 3. Приведение к другому типу  
# Scale сохранит нормализованные данные. Без него img будет иметь min 0 и max 255  
transform = v2.ToDtype(torch.float32, scale=True)  
  
img_dtype_v2 = transform(img_v2)  
  
# Трансформация 4. Нормализация  
  
transform = v2.Normalize(mean=(0.5, 0.5, 0.5), std=(0.5, 0.5, 0.5))  
  
img_norm_v2 = transform(img_dtype_v2)  
  
# Compose  
  
transform = v2.Compose(  
    [  
        v2.ToImage(),  
        v2.ToDtype(torch.float32, scale=True),  
        v2.Normalize(mean=(0.5, 0.5, 0.5), std=(0.5, 0.5, 0.5))  
    ]  
)
```