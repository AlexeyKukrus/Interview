**Виртуализация (или ленивый рендеринг)** во Frontend-разработке — это техника оптимизации, которая используется для работы с большими объемами данных или элементов интерфейса. Она позволяет рендерить только те элементы, которые видимы на экране пользователя, вместо того чтобы отрисовывать всё сразу.

***Виртуализация решает проблемы:***
	1. Проблема высокой нагрузки браузера
	2. Проблему фриза UI, что ухудшает отзывчивость

```jsx
import { useState, useRef, useEffect } from 'react';

function VirtualizedList({ items, itemHeight, containerHeight }) {
  // Храним индекс первого видимого элемента
  const [startIndex, setStartIndex] = useState(0);

  // Ссылка на контейнер, чтобы отслеживать прокрутку
  const containerRef = useRef(null);

  // Сколько элементов можно отобразить на одной странице (в зоне видимости)
  const itemsPerPage = Math.ceil(containerHeight / itemHeight);

  // Общая высота списка для правильной прокрутки
  const totalHeight = items.length * itemHeight;

  // Обработчик события прокрутки
  const handleScroll = () => {
    // Текущее положение прокрутки
    const scrollTop = containerRef.current.scrollTop;
    // Вычисляем индекс первого видимого элемента
    const newStartIndex = Math.floor(scrollTop / itemHeight);
    setStartIndex(newStartIndex);
  };

  // Добавляем обработчик прокрутки при монтировании компонента
  useEffect(() => {
    const container = containerRef.current;
    container.addEventListener('scroll', handleScroll);
    // Убираем обработчик при размонтировании компонента
    return () => container.removeEventListener('scroll', handleScroll);
  }, []);

  // Получаем элементы, которые видимы в данный момент
  const visibleItems = items.slice(startIndex, startIndex + itemsPerPage);

  return (
    <div
      ref={containerRef}
      style={{
        height: `${containerHeight}px`,
        overflow: 'auto', // Контейнер прокручивается
        position: 'relative', // Для позиционирования внутренних элементов
      }}
    >
      {/* Невидимый заполнитель с полной высотой списка */}
      <div
        style={{
          height: `${totalHeight}px`,
          position: 'relative', // Для абсолютного позиционирования видимых элементов
        }}
      >
        {visibleItems.map((item, index) => (
          <div
            key={startIndex + index} // Уникальный ключ для каждого элемента
            style={{
              position: 'absolute', // Абсолютное позиционирование внутри списка
              top: `${(startIndex + index) * itemHeight}px`, // Смещение элемента
              height: `${itemHeight}px`, // Фиксированная высота элемента
              width: '100%', // Полная ширина контейнера
            }}
          >
            {item} {/* Содержимое элемента списка */}
          </div>
        ))}
      </div>
    </div>
  );
}

// Пример использования VirtualizedList
function App() {
  // Создаем массив с 10,000 элементов
  const items = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`);

  // Отображаем виртуализированный список
  return <VirtualizedList items={items} itemHeight={30} containerHeight={500} />;
}


```