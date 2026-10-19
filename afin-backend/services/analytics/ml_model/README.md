# ML Model Files

Эта директория должна содержать файлы ML модели для предсказания задержек процессов.

## Необходимые файлы

Скопируйте следующие файлы из `ai-service/best_model_LightGBM/` в эту директорию:

1. `delay_predictor.txt` - обученная модель LightGBM
2. `scaler.pkl` - scaler для нормализации признаков
3. `feature_names.json` - список признаков модели

## Инструкция по копированию

### Windows (PowerShell)
```powershell
Copy-Item "..\..\..\..\..\ai-service\best_model_LightGBM\*" -Destination "." -Recurse
```

### Windows (CMD)
```cmd
xcopy /E /I /Y "..\..\..\..\..\ai-service\best_model_LightGBM\*" .
```

### Linux/Mac
```bash
cp -r ../../../../ai-service/best_model_LightGBM/* .
```

## Проверка

После копирования в директории должны быть:
- `delay_predictor.txt`
- `scaler.pkl`
- `feature_names.json`

Если файлы отсутствуют, сервис выдаст ошибку при попытке загрузить модель.





