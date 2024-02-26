# Решение для соревнования Alfa Campus Income prediction в рамках проекта Alfa Campus   
Название команды - SYDE  
[Ссылка на соревнование kaggle](https://www.kaggle.com/competitions/income-prediction-alfa-campus) 
## Структура решения  
### 01_eda_baseline: 
В данном файле проводится предварительная обработка данных, проверка и выдвижение различных гипотез, обучение baseline-модели.  
### 02_train_model_w: 
В файле *01_eda_baseline* выдвинута гипотеза о том, что модельное предсказание и добавление в качестве фичи веса, который используется для расчета метрики, может повысить качество итоговой модели. В данном файле обучается модель, которая предсказывает вес.  
### 03_ensemble_model_w: 
В данном файле предсказанный вес используется в качестве фичи итогового регрессора. Выяснено, что при таком подходе модель сильно переобучается, поэтому принято решение отложить его.   
### 04_train_clf: 
В файле *01_eda_baseline* было выяснено, что модель занижает прогноз для клиентов с высокой зарплатой и завышает для клиентов с низкой зарплатой. Выдвинута гипотеза, что предсказание сегмента доходности клиента может улучшить качество итогового регрессора.  
  
![residual plot](https://github.com/koshkidadanet/income-prediction-alfa-campus/assets/56166716/ce9f928e-a430-47af-86e8-c2477c3b5813)  
  
Выделено 2 подхода:  
- Использование предсказаний классификатора в качестве фичей для регрессора
- Построение по одному регрессору на каждый предсказанный класс  
  
В данном файле обучаются классфикаторы. Эксперементы проводились с разделением клиентов на 2 и на 3 сегмента доходности.  
### 05_ensemble_model_clf_f: 
В данном файле предсказания классификатора используются как фичи регрессора. Такой подход позволил существенно улучшить метрику на валидационном и публичном датасетах, поэтому принято решение развивать идею с использованием классификатора.  
### 06_ensemble_3model_clf: 
В данном файле обучается 3 регрессора для каждого сегмента, предсказанного классифкатором. Такой подход позволил улучшить метрику, но возник вопрос об оптимальных границах сегментов доходности. В качестве бэйзлайна все клиенты были разбиты на 3 равных по количеству сегмента.    
Оптимальные границы для регрессора проанализированы в файле *01_eda_baseline*, пункт *Улучшиться ли качество модели, если разбить датасет на бины по таргету?* Выяснено, что оптимальными границами сегментов для регроссора являются 0.7 и 0.9 квантиль по таргету. Но с такими границами метрика на публичном датасете ухудшалась. Даная проблема была проанализирована в файле *08_ensemble_analysis.ipynb*  
### 07_ensemble_2model_clf: 
Протестирован такой же подход, что и в файле *06_ensemble_3model_clf*, но с бинарныйм классифкатором на 2 сегмента доходности. Метрика на публичном и валидационном датасете оказалась лучше при разбиении на 3 сегмента доходности, поэтому далее использовалось только 3 сегмента.  
### 08_ensemble_analysis: 
В данном файле анализируется, почему при обучении пайплайна на оптимальных для регрессора границах (0.7, 0.9 квантили), ухудшается метрика на публичном датасете. Выяснено, что при использовании таких границ, регроссоры сильно переобучаются, особенно регрессор, отвечающий за клиентов с экстримально высоким доходом. Принято решение разделять датасет на 3 равные сегменты, то есть границы будут 1/3 и 2/3 квантили.   
  
*Переобучение регрессора на втором сегменте (экстримально высокий доход)*  
![2segm_07_09](https://github.com/koshkidadanet/income-prediction-alfa-campus/assets/56166716/f52c4c54-a2d1-4395-9326-b39126889ec8)
  
### 09_feature_engineering:  
В данном файле объеденины все командные подходы по генерации дополнительных фичей, включая фичи из внешних источников, и сохраняется итоговый датаесет.  
### 10_tuning_clf_kgl:  
Файл для обучения классификатора на kaggle с использованием GPU. В данном файле добавлен алгоритм отбора фичей RecursiveByShapValues и подбор гиперпараметров с помощью optuna.  
### 11_tuning_reg_kgl:  
Файл для обучения регрессора с фичами классифкатора (первый подход) на kaggle с использованием GPU. В данном файле добавлен алгоритм отбора фичей RecursiveByShapValues и подбор гиперпараметров с помощью optuna.  
### 12_ensemble_clf_inference:  
Файл для итоговой разметке тестового датасета на моделях, обученных на kaggle  
## Идеи для развития  
* Найти оптимальные границы сегментов для всего пайплайна. Границы 0.7 и 0.9 были найдены только для регрессора, а на всем пайплайне они показали результат хуже, чем границы 1/3 и 2/3.
* Развитие подхода с тремя регрессорами на предсказаниях классификатора. Возможно метрика улучшится, если для каждого регрессора сделать индивидуальные отбор фичей и подбор гиперпараметоров.
* При обучению регрессоров отдельно на каждый сегмент, обогощать их обучающую выборку данными из других сегментов. Возможно это снизит вероятность переобучения и улучшит итоговую метрику.
