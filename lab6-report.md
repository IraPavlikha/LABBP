# Лабораторна робота 6. Реалізація розширених можливостей та деплой frontend

## Мета роботи
Здобути практичні навички впровадження real-time функціональності, створення розширених UI компонентів, оптимізації продуктивності та деплою React застосунку на хостинг платформу.

## Завдання

1. Налаштувати Socket.io-client та реалізувати real-time сповіщення.

2. Створити модальні вікна, таби та dropdown меню.

3. Реалізувати drag and drop, графіки, infinite scroll та image upload з preview.

4. Впровадити lazy loading, PWA з offline режимом, оптимізувати продуктивність.

5. Деплоїти frontend на Vercel/Netlify та налаштувати CI/CD.

---
## Хід роботи
###  Крок 1. Налаштування Socket.io-client

```typescript
import { io, Socket } from 'socket.io-client';

class SocketService {
  private socket: Socket | null = null;

  connect(token: string) {
    this.socket = io(import.meta.env.VITE_API_URL, {
      auth: { token },
      transports: ['websocket'],
    });
    this.socket.on('connect', () => console.log('WebSocket connected'));
    return this.socket;
  }

  disconnect() {
    this.socket?.disconnect();
    this.socket = null;
  }

  on(event: string, callback: (...args: any[]) => void) {
    this.socket?.on(event, callback);
  }

  emit(event: string, data?: any) {
    this.socket?.emit(event, data);
  }
}

export default new SocketService();

```
### Крок 2. Реалізація real-time сповіщень
```typescript
import { useEffect } from 'react';
import { useSocket } from '../hooks/useSocket';
import { useNotificationsStore } from '../stores/notificationsStore';

const NotificationBell = () => {
  const socket = useSocket();
  const { notifications, addNotification } = useNotificationsStore();

  useEffect(() => {
    socket.on('notification', (data) => {
      addNotification({ type: data.type, title: data.title, message: data.message });
    });
    return () => socket.off('notification');
  }, [socket, addNotification]);

  return <div> {/* UI сповіщень */} </div>;
};

```
### Крок 3. Модальні вікна
```typescript
import { Dialog, Transition } from '@headlessui/react';

const Modal = ({ isOpen, onClose, title, children }) => (
  <Transition appear show={isOpen} as={Fragment}>
    <Dialog as="div" onClose={onClose}>
      <Dialog.Panel>{children}</Dialog.Panel>
    </Dialog>
  </Transition>
);
```
### Крок 4. Drag and Drop
```typescript
import { DndContext, useSensor, useSensors, PointerSensor } from '@dnd-kit/core';
import { arrayMove, SortableContext, verticalListSortingStrategy } from '@dnd-kit/sortable';

<DndContext sensors={useSensors(useSensor(PointerSensor))} onDragEnd={handleDragEnd}>
  <SortableContext items={tasks.map(t => t.id)} strategy={verticalListSortingStrategy}>
    {tasks.map(task => <SortableItem key={task.id} id={task.id}>{task.title}</SortableItem>)}
  </SortableContext>
</DndContext>

```
### Крок 5. Графіки
```typescript
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip } from 'recharts';

const StatsChart = ({ data }) => (
  <LineChart data={data}>
    <CartesianGrid strokeDasharray="3 3" />
    <XAxis dataKey="name" />
    <YAxis />
    <Tooltip />
    <Line type="monotone" dataKey="value" stroke="#3B82F6" />
  </LineChart>
);

```
### Крок 6. Lazy Loading
```typescript
const HomePage = React.lazy(() => import('./pages/HomePage'));
<Suspense fallback={<Loading />}>
  <Routes>
    <Route path="/" element={<HomePage />} />
  </Routes>
</Suspense>

```
### Крок 7. Налаштування PWA
```typescript
import { VitePWA } from 'vite-plugin-pwa';

VitePWA({
  registerType: 'autoUpdate',
  manifest: { name: 'My React App', short_name: 'MyApp', icons: [{ src: 'pwa-192x192.png', sizes: '192x192', type: 'image/png' }] },
});
```

### Крок 8. Деплой на Vercel

Підключення GitHub репозиторію

Налаштування environment variables

Автоматичний деплой при push до main branch

## Висновки

В результаті виконання лабораторної роботи було досягнуто наступного:

- Реалізовано real-time функціональність через Socket.io.

- Створено інтерактивні компоненти: модалки, таби, dropdown, drag and drop, графіки.

- Оптимізовано продуктивність за допомогою lazy loading, React.memo, PWA та кешування.

- Проведено деплой фронтенду на Vercel та налаштовано CI/CD.

Застосунок відповідає сучасним вимогам: responsive дизайн, offline підтримка, швидке завантаження та SEO оптимізація.
---

## Відповіді на контрольні запитання

1. **Різниця між WebSocket та HTTP протоколами. Коли доцільно використовувати WebSocket**

- HTTP — однонаправлений протокол, клієнт відправляє запит → сервер відповідає. Підходить для стандартних CRUD операцій.

- WebSocket — двостороннє постійне з’єднання, сервер може надсилати дані без запиту клієнта. Підходить для чатів, live сповіщень, онлайн ігор, collaborative editing та live дашбордів.
Приклад використання Socket.io:
```typescript
import { io } from 'socket.io-client';
const socket = io('http://localhost:3000');
socket.on('message', data => console.log(data));
```

2. **Управління життєвим циклом WebSocket у React компонентах**
Використовують useEffect для ініціалізації та закриття з’єднання, щоб уникнути витоків пам’яті.
```typescript
useEffect(() => {
  socket.connect();
  return () => {
    socket.disconnect();
  };
}, []);
```

3. **React портали та модальні вікна**
React Portal дозволяє рендерити компонент поза DOM-ієрархією батьківського елемента, що корисно для модальних вікон, tooltip, dropdown.
```typescript
import { createPortal } from 'react-dom';
const Modal = ({ children }) => {
  return createPortal(
    <div className="modal">{children}</div>,
    document.getElementById('modal-root')
  );
};
```

4. **Порівняння бібліотек для drag and drop у React. Переваги dnd-kit**

- react-dnd — потужна, декларативна, складна у налаштуванні, потребує багато boilerplate.

- react-beautiful-dnd — простіше для списків, але не підтримує багатоплатформові сценарії.

- dnd-kit — легка, сучасна, висока продуктивність, accessibility за замовчуванням, гнучкі drop zones, підтримка мобільних пристроїв.

5. **Code splitting та React.lazy()**

Code splitting — розбиття JavaScript bundle на менші частини, що завантажуються за потребою.

React.lazy() дозволяє динамічно імпортувати компонент, React під капотом створює Suspense boundary, поки компонент завантажується.
```typescript
const Dashboard = React.lazy(() => import('./Dashboard'));
<Suspense fallback={<Loading />}>
  <Dashboard />
</Suspense>
```

6. **Service Worker та offline функціональність PWA**
Service Worker — скрипт, що працює у фоновому режимі, перехоплює мережеві запити, кешує ресурси, забезпечує offline режим.
Приклад:
```typescript
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(response => response || fetch(event.request))
  );
});
```

7. **Метрики продуктивності вебзастосунків**

- FCP (First Contentful Paint) — час до відображення першого контенту

- TTI (Time to Interactive) — час до повної інтерактивності

- CLS (Cumulative Layout Shift) — стабільність верстки

- LCP (Largest Contentful Paint) — час рендерингу найбільшого елементу
Вимірюються через Lighthouse, Web Vitals, Chrome DevTools.
Покращення: lazy loading, code splitting, оптимізація зображень, memoization компонентів, caching.

8. **Vercel vs Netlify для деплою React застосунків**

Vercel: оптимізований для Next.js, швидкий автоматичний деплой, serverless functions, глобальна CDN.

Netlify: простий для статичних сайтів, додаткові сервіси (форми, identity), також має CI/CD.
Відмінність: Vercel краще для SSR, Netlify для JAMstack і SPA.

9. **CI/CD та переваги автоматизації деплою**

- CI (Continuous Integration) — автоматичне тестування та збірка при push

- CD (Continuous Deployment) — автоматичний деплой після успішної збірки
Переваги: швидкий реліз, зменшення людських помилок, покращення стабільності та якості продукту.

10. **SEO оптимізація для Single Page Applications**

- Використання meta tags, Open Graph, structured data

- Серверний рендеринг (SSR) або pre-rendering (наприклад, Vite SSR)

- Sitemap.xml та robots.txt

- Використання semantic HTML, lazy loading із правильними атрибутами, title та description для кожної сторінки
