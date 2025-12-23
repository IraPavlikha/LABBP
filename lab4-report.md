# Лабораторна робота 4. Реалізація React проєкту
## Клієнтська частина системи онлайн-аукціонів
## Огляд проєкту

Проєкт “AuctionApp” — це клієнтська частина платформи для онлайн-аукціонів антикваріату. Користувачі можуть переглядати лоти, фільтрувати їх за категоріями, переглядати деталі лотів та взаємодіяти з backend через HTTP-запити.  

Мета лабораторної роботи — створити сучасний React-проєкт з TypeScript, налаштувати базову архітектуру, UI-компоненти та інтегрувати frontend із backend.

## Мета лабораторної роботи

- Створити React проєкт з Vite та TypeScript.  
- Налаштувати Tailwind CSS для стилізації компонентів.  
- Розробити базові UI-компоненти (Button, Input, Card, Badge).  
- Реалізувати навігацію за допомогою React Router.  
- Інтегрувати Axios для взаємодії з backend та налаштувати interceptors.  
- Впровадити responsive дизайн та темну тему через Tailwind.  

## Технологічний стек

| Компонент | Технологія |
|-----------|------------|
| Frontend | React 18, TypeScript, Tailwind CSS |
| Routing | React Router v6 |
| HTTP запити | Axios |
| Стилізація | Tailwind CSS |
| Лінтинг та форматування | ESLint + Prettier |
| Конфігурації | Environment variables (.env) |
| Архітектура | Компонентна, Layout компоненти, переусні UI елементи |

## Архітектура системи

Клієнтська частина побудована на принципах компонентної архітектури:

- **UI компоненти:** Button, Input, Card, Badge.  
- **Layout компоненти:** Header, Footer, Layout.  
- **Сторінки:** Home, About, NotFound.  
- **Сервіси:** Axios для API-запитів.  
- **Типи:** TypeScript інтерфейси для API відповідей та пропсів компонентів.  
- **Утиліти:** функції для форматування даних, кастомні hook-и.  

## Функціонал

- Перегляд усіх лотів у вигляді карток.  
- Навігація між сторінками Home та About.  
- Базова авторизація через токен (JWT).  
- Адаптивний дизайн для різних екранів.  
- Темна тема через Tailwind dark mode.  
- Axios interceptors для обробки запитів та помилок.

### Приклад базового UI-компонента (Button)

```typescript
import React from 'react';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
}

const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'md',
  isLoading = false,
  className = '',
  ...props
}) => {
  const baseClasses = 'font-medium rounded transition-colors duration-200';

  const variantClasses = {
    primary: 'bg-blue-600 hover:bg-blue-700 text-white',
    secondary: 'bg-gray-200 hover:bg-gray-300 text-gray-800',
    danger: 'bg-red-600 hover:bg-red-700 text-white',
  };

  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2',
    lg: 'px-6 py-3 text-lg',
  };

  return (
    <button
      className={`${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${className}`}
      disabled={isLoading}
      {...props}
    >
      {isLoading ? 'Завантаження...' : children}
    </button>
  );
};

export default Button;
```
### Layout компонент (Header + Footer) 

```
import React from 'react';
import { Link } from 'react-router-dom';

const Header: React.FC = () => (
  <header className="bg-white shadow-md">
    <nav className="container mx-auto px-4 py-4 flex justify-between items-center">
      <Link to="/" className="text-2xl font-bold text-blue-600">AuctionApp</Link>
      <div className="space-x-6">
        <Link to="/" className="hover:text-blue-600">Головна</Link>
        <Link to="/about" className="hover:text-blue-600">Про нас</Link>
      </div>
    </nav>
  </header>
);

export default Header;
```
### Environment variables

```
VITE_API_URL=http://localhost:3000/api
VITE_APP_NAME=AuctionApp
```

### Структура проєкту
```
src/
├── components/      # Переусні UI компоненти
│   ├── common/     # Button, Input, Card
│   └── layout/     # Header, Footer, Layout
├── pages/          # Home, About, NotFound
├── services/       # Axios конфігурація
├── types/          # TypeScript типи
├── utils/          # Утиліти та helper-и
├── hooks/          # Кастомні React hook-и
└── constants/      # Константи застосунку
```

## Висновок
Виконання лабораторної роботи дозволило здобути практичні навички створення React-проєкту з TypeScript, Tailwind CSS та базовою архітектурою frontend-застосунку. Були налаштовані routing, Axios для API, базові UI-компоненти та темна тема. Проєкт готовий до подальшої розробки функціональних можливостей та інтеграції з backend.

### 1. Які переваги має Vite порівняно з Create React App? Поясніть концепцію ES modules у браузері.

Vite запускається набагато швидше, ніж CRA, завдяки використанню нативних ES модулів у браузері. Він не збирає весь код під час розробки, а завантажує модулі на вимогу. ES modules (ESM) дозволяють браузеру імпортувати та виконувати окремі JS-файли без bundler-ів, що пришвидшує роботу і полегшує HMR (hot module replacement).

### 2. Що таке TypeScript інтерфейси та як вони використовуються для типізації пропсів компонентів?

Інтерфейси в TypeScript — це контракти для об’єктів, які визначають їхні властивості та типи. У React їх використовують для типізації пропсів компонентів, щоб забезпечити перевірку даних під час компіляції і автодоповнення в IDE.
Приклад:
```
interface ButtonProps {
  label: string;
  onClick: () => void;
}
const Button: React.FC<ButtonProps> = ({ label, onClick }) => <button onClick={onClick}>{label}</button>;
```
### 3. Поясніть концепцію utility-first CSS та основні переваги Tailwind CSS.

Utility-first CSS — це підхід, коли стилі будуються через маленькі класи утиліт (наприклад, p-4, text-center). Tailwind CSS дозволяє швидко створювати адаптивні та консистентні інтерфейси без написання кастомного CSS, легко змінювати теми та використовувати dark mode.

### 4. Як працюють interceptors у Axios? Наведіть приклади їх використання.

Axios interceptors — це функції, які обробляють запити або відповіді глобально, до того як вони потраплять у компонент.
Приклади:

Додавання JWT токена до заголовків:
```
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if(token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

Обробка помилок 401:
```
api.interceptors.response.use(
  res => res,
  error => {
    if(error.response?.status === 401) localStorage.removeItem('token');
    return Promise.reject(error);
  }
);
```
### 5. Що таке React Router Outlet та як він використовується для вкладених маршрутів?

Outlet — компонент, який рендерить дочірні маршрути всередині Layout-компонента. Він дозволяє вкладати сторінки у спільний шаблон (Header/Footer) без дублювання коду.

### 6. Поясніть різницю між ESLint та Prettier. Як уникнути конфліктів між ними?

ESLint — перевіряє код на помилки та дотримання стандартів.

Prettier — автоматично форматує код (відступи, лапки, розбиття рядків).
Конфлікти уникати допомагає eslint-config-prettier, який вимикає правила ESLint, що можуть конфліктувати з Prettier.

### 7. Що таке environment variables та чому важливо не комітити файл .env до репозиторію?

Environment variables — змінні, які зберігають конфіденційні дані (API ключі, URL серверів). .env не слід комітити, щоб уникнути витоку секретів у публічний репозиторій.

### 8. Які принципи слід дотримуватися при створенні переусних компонентів?

Один компонент = одна відповідальність.

Використання пропсів для передачі даних.

Повторне використання і легке тестування.

DRY (не дублювати код).

Структурувати компоненти логічно (common, layout, pages).

### 9. Поясніть поняття responsive дизайну та як Tailwind спрощує його реалізацію.

Responsive дизайн забезпечує коректне відображення на різних пристроях. Tailwind надає breakpoint-класи (sm:, md:, lg:), щоб легко задавати стилі під різні розміри екранів без додаткового CSS.

### 10. Як організувати структуру папок у великому React проєкті? Які підходи ви знаєте?

components/ — переусні UI-компоненти (common, layout).

pages/ — сторінки (Home, About, NotFound).

services/ — API запити та Axios конфігурація.

types/ — TypeScript інтерфейси та типи.

utils/ — допоміжні функції.

hooks/ — кастомні React hook-и.

constants/ — константи застосунку.
Підхід: логічне розділення за функціоналом, щоб легко масштабувати та підтримувати код.
