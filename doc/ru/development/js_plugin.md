# Плагин Face SDK VideoEngine JS

Плагин **Face SDK VideoEngine JS** реализует функции [детекции и трекинга лиц](/doc/ru/development/face_capturing.md) и определения принадлежности лица живому человеку (Active Liveness). В данном разделе представлена информация по установке, инициализации и запуску плагина **Face SDK VideoEngine JS**, доступным параметрам запуска и публичным методам.

## Установка плагина 

Выполните одну из следующих команд в консоли, в зависимости от платформы, которую Вы используете (Node или Yarn): 
* `npm install @3divi/face-sdk-video-engine-js`
* `yarn add @3divi/face-sdk-video-engine-js`

## Инициализация плагина 

1. Импортируйте библиотеку `VideoEngine` из Face SDK:  
`import { VideoEngine } from '@3divi/face-sdk-video-engine-js';`
2. Создайте экземпляр класса `VideoEngine` (см. описание опциональных параметров запуска в пункте [Параметры запуска плагина](js_plugin.md#параметры-запуска-плагина)):   
`const videoEngine = new VideoEngine(?options);` 

## Запуск плагина 

1. Выполните вызов метода `load` для загрузки модуля: `videoEngine.load();`
2. Для запуска плагина вызовите метод `videoEngine.start(input, callback);`, где:
   * `input` — HTML-элемент `HTMLVideoElement`, который можно получить из тэга `<video></video>` и передать его в качестве `input` в метод. Внутри тэга должен быть указан видеопоток (*video stream*) (см. пункт [Инициализация камеры](js_plugin.md#инициализация-камеры))
   * `callback` — функция обратного вызова, в которой можно получить объект `prediction` (см. описание объекта в пункте [Публичные методы](js_plugin.md#публичные-методы)) и обрабатывать данные в реальном времени во время процессинга (режим, в котором происходит обработка видеопотока с камеры: детекция лица, определение Liveness), например, отобразить ограничивающий прямоугольник, получить данные и вывести их на экран.

_**Примечание**: необходимо учитывать, что загрузка модуля — это асинхронный процесс._

Пример инициализации плагина: 
```
const initVideoEngine = async () => {
 try {
  // Do something
  const videoEngine = new VideoEngine();
  await videoEngine.load();
  // Do something
 } catch (error) {
   console.log(error.message);
 }
};
```

### Инициализация камеры

Для инициализации камеры необходимо:
1. Добавить в HTML тэг `<video></video>`
2. Получить тэг видео:  
`const video = documnet.querySelector('video');`
3. Инициализировать поток (`stream`). Полный пример: [demo/src/utils.js](/examples/javascript/demo/src/utils.js).
```
try {
  const stream = await navigator.mediaDevices.getUserMedia({
   audio: false,
   video: {
    facingMode: 'user',
    width: VIDEO_SIZE,
    height: VIDEO_SIZE,
   },
  });
  video.srcObject = stream;
  return new Promise((resolve) => {
   video.onloadedmetadata = () => {
    resolve(video);
   };
  });
 } catch (error) {
  throw new Error(error);
}
```

### Параметры запуска плагина

Функция `new VideoEngine()` принимает опциональный объект `options`, который содержит следующие поля:
* `backend` — используемое средство для обработки видеопотока. Значения: `webgl` — для обработки используется видеокарта GPU, `cpu` — для обработки используется ЦП. Значение по умолчанию: `webgl`
* `pose` — объект, хранящий в себе поворот головы в градусах. Используется для проверки Активного Liveness и ограничения максимально разрешенного угла поворота головы (см. [Рекомендации по размещению камер и съемке](/doc/ru/guidelines_for_cameras.md#размещение-камер-и-съемка)). Имеет поле `maxRotation`. Принимает значение типа *Number* (градус поворота). Значение по умолчанию: `15` 
* `eyes` — объект, хранящий в себе информацию о положении и состоянии глаз. Принимает следующие поля:
    * `minDistance` — минимальное расстояние между зрачками в пикселях. Тип: *Number*. Значение по умолчанию: `60`. Также возможно импортировать константы для последующей передачи:
        * `REGISTRATION_EYES_MIN_DISTANCE` хранит значение `60`. Лицо добавляется в базу, если удовлетворено данное значение. 
        * `IDENTIFICATION_EYES_MIN_DISTANCE` хранит значение `40`. Лицо идентифицируется, если удовлетворено данное значение. 
    * `closeLowThreshold` — минимальное пороговое значение для состояния “глаза открыты” (от 0 до 1). Если значение меньше порога `closeLowThreshold`, результатом является статус “глаза закрыты”. Принимает значение типа *Number* (от 0 до 1). Значение по умолчанию: `0.25`.
    * `closeHighThreshold` — максимальное пороговое значение для состояния “глаза открыты”. Если глаза закрыты, используется значение `closeHighThreshold`. Если значение >= `closeHighThreshold`, результатом является статус “глаза открыты”, иначе — “глаза закрыты”. Принимает значение типа *Number* (от 0 до 1). Значение по умолчанию: `0.3`. Если значение `closeHighThreshold` не указано, оно высчитывается автоматически в зависимости от `closeLowThreshold`.
    * `maxDurationClose` — значение, используемое для определения действия “моргание”. Если промежуток времени между состояниями “глаза открыты” и “глаза закрыты” меньше значения `maxDurationClose`, результатом является статус “было произведено моргание”. Принимает значение типа *Number* (время в мс). Значение по умолчанию: `500`.

## Публичные методы

* `load` — асинхронная инициализация библиотеки. Может принимать поле `backend` (см. [Параметры запуска плагина](js_plugin.md#параметры-запуска-плагина)). Возвращает объект `Promise`. В результате завершения вызова приходит объект `Status`: 
	```
	{
		type: 'success',
		message: 'ok',
	};
	```
* `reset` — полная очистка данных в библиотеке о последнем процессинге.
* `start` — запускает обработку видеопотока. Возвращает объект `Promise`, который содержит в себе объект `Status` (см. выше). При первом выполнении метода `start` происходит инициализация модели и объекта `backend`. Принимает следующие параметры:
    * `input` — HTML-элемент `HTMLVideoElement` (см. [Запуск плагина](js_plugin.md#запуск-плагина)).
    * `callback` — функция обратного вызова, внутри которой в режиме реального времени можно обрабатывать получаемые данные. Аргумент данной функции — объект `prediction`.
        * `prediction`:
            * `pose` — объект, хранящий информацию о текущем положении головы:
                * `axes` — объект связанной системы координат *roll/pitch/yaw*, поля которого имеют тип *Number*.
                * `poseInRequiredRange` — переменная, означающая, что положение головы находится в корректном диапазоне связанной системы координат (углы наклона не превышают значение `maxRotation`). Тип: *Boolean*. 
            * `headWasTurned` — была ли повернута голова во время процессинга. Тип: *Boolean*.
            * `imageBase64` — изображение текущего объекта `prediction`. Тип: *String*, формат данных: *Base64*.
            * `face` — объект, который содержит информацию о глазах и точках лица:
                * `boundingBox` — объект ограничивающего прямоугольника лица:
                    * `topLeft` — массив 2D координат [x:number, y:number]; (верхний левый угол).
                    * `bottomRight` — массив 2D координат [x:number, y:number]; (нижний правый угол). 
                * `faceInViewConfidence` — вероятность присутствия лица. Тип: *Number* (от 0 до 1).
                * `mesh` — массив с вложенными массивами 3D координат точек лица [...[x:number, y:number, z:number]].
                * `scaledMesh` — нормализированный массив (координаты точек скорректированы в соответствии с размером лица на видеопотоке) с вложенными массивами 3D координат точек лица [...[x:number, y:number, z:number]].
                * `annotations` — семантические группы координат `scaledMesh`.
                * `eyes` — объект, в котором хранится информация о глазах:
                    * `isClosed` — закрыты ли глаза в текущий момент времени. Тип: *Boolean*.
                    * `isBlinked` — было ли произведено моргание в текущий момент времени. Тип: *Boolean*.
                    * `wasBlinked` — было ли произведено моргание в течение всего периода процессинга. Тип: *Boolean*.
                    * `eyesDistanceInRequiredRange` — находится ли пользователь на корректном растоянии до камеры. При расчете учитывается значение параметра `minDistance` (минимальное расстояние между зрачками). Тип: *Boolean*.
                    * `levelOpeningEyes` — объект, хранящий текущую степень открытия каждого глаза:
                        * `left` — левый глаз. Тип: *Number*.
                        * `right` — правый глаз. Тип: *Number*.
    * Поскольку первая инициализация после вызова метода `start` может занимать много времени, Вы можете дополнить код вызова, чтобы в процессе инициализации отображалось сообщение (например, “Происходит инициализация”).
* `stop` — останавливает процесс обработки, но сохраняет текущие значения. В случае вызова метода `start` обработка продолжится. Возвращает лучший объект `prediction`.
   
Пример интерфейса, возвращаемого в объекте `prediction`:
```
interface OutputData {
  pose: {
    axes: Axes;
    poseInRequiredRange: boolean;
  };
  face: {
    boundingBox: BoundingBox;
    mesh: Coord3D[];
    scaledMesh: Coord3D[];
    annotations?: { [key: string]: Coord3D };
    faceInViewConfidence: number;
    eyes: {
        isClosed: boolean;
        isBlinked: boolean;
        wasBlinked: boolean;
        eyesDistanceInRequiredRange: boolean;
        levelOpeningEyes: {
          left: number;
          right: number;
        };
  };
};
  headWasTurned: boolean;
  imageBase64?: string;
}
```

## Публичные поля

* `backend` — используемое средство для процессинга (см. [Параметры запуска плагина](js_plugin.md#параметры-запуска-плагина)). Возвращает данные типа *String*.
* `inProgress` — был ли запущен процессинг. Возвращает булево значение.
* `isInitialised` — была ли инициализирована модель. Инициализация модели происходит во время первого выполнения метода `start`. Возвращает булево значение.
* `isLoaded` —  была ли загружена модель. Возвращает булево значение.
* `bestPrediction` — возвращает лучший объект `prediction`.
* `bestShot` — возвращает лучшее изображение лучшего объекта `prediction`.