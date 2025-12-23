# Лабораторна робота 5. Розроблення основного інтерфейсу та функціоналу

## Мета роботи
Здобути практичні навички створення повнофункціонального користувацького інтерфейсу з інтеграцією з backend API, системою аутентифікації, управлінням станом, валідацією форм та реалізацією CRUD операцій.

## Завдання
Створити клієнтську частину застосунку для роботи з аукціонами, користувачами та продуктами:

1. **Система аутентифікації**  
   - Реалізація сторінок Login та Register.  
   - Збереження JWT токена після входу користувача.  
   - Захист маршрутів через Protected Routes.  
   - Можливість logout та очищення токена.  

2. **Dashboard та навігація**  
   - Головна сторінка з блоками для переходу до сутностей: користувачі, продукти, аукціони.  

3. **CRUD операції**  
   - Створення, редагування, перегляд та видалення записів.  
   - Валідація форм через React Hook Form + Zod.  
   - Підтвердження дій при видаленні.  

4. **UI/UX покращення**  
   - Toast сповіщення для success/error.  
   - Пошук та фільтрація даних.  
   - Skeleton loaders для покращення UX під час завантаження.  
   - Responsive дизайн із sidebar, що згортається на мобільних пристроях.  

5. **Управління станом**  
   - Використання Zustand для глобального стану аутентифікації та списків сутностей.  

---
## Основні реалізації

### Аутентифікація користувача
- Збереження токена та даних користувача у Zustand store.
- ProtectedRoute для захисту сторінок Dashboard і CRUD компонентів.

### CRUD операції
- Сервіси для роботи з API (`usersService`, `auctionService`).  
- Store для стану списку сутностей (`usersStore`, `auctionsStore`).  
- Сторінки: список, створення, редагування, перегляд.  
- Валідація через Zod.  

### Покращення UX
- Toast повідомлення через `react-hot-toast`.  
- Skeleton loaders під час завантаження списків.  
- Пошук та фільтрація у списках сутностей.  

---
## Лістинги коду

### 1. Контролери аукціонів (`auction.controller.js`)
```javascript
import uploadImage from '../services/cloudinaryService.js';
import Product from '../models/product.js';
import mongoose from "mongoose"
import { connectDB } from '../connection.js'

export const createAuction = async (req, res) => {
    try {
        await connectDB();
        const { itemName, startingPrice, itemDescription, itemCategory, itemStartDate, itemEndDate } = req.body;
        let imageUrl = '';

        if (req.file) {
            try {
                imageUrl = await uploadImage(req.file);
            } catch (error) {
                return res.status(500).json({ message: 'Error uploading image to Cloudinary', error: error.message });
            }
        }

        const start = itemStartDate ? new Date(itemStartDate) : new Date();
        const end = new Date(itemEndDate);
        if (end <= start) {
            return res.status(400).json({ message: 'Auction end date must be after start date' });
        }

        const newAuction = new Product({
            itemName,
            startingPrice,
            currentPrice: startingPrice,
            itemDescription,
            itemCategory,
            itemPhoto: imageUrl,
            itemStartDate: start,
            itemEndDate: end,
            seller: req.user.id,
        });
        await newAuction.save();

        res.status(201).json({ message: 'Auction created successfully', newAuction });
    } catch (error) {
        res.status(500).json({ message: 'Error creating auction', error: error.message });
    }
};
```
### 2. Сервіс завантаження зображень (cloudinaryService.js)
```javascript
import cloudinary from 'cloudinary';
import dotenv from 'dotenv';
dotenv.config().parsed;

cloudinary.config({
    cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
    api_key: process.env.CLOUDINARY_API_KEY,
    api_secret: process.env.CLOUDINARY_API_SECRET,
});

const uploadImage = async (file) => {
    try {
        const result = await cloudinary.uploader.upload(file.path, {
            folder: 'auctions',
        });
        return result.secure_url;
    } catch (error) {
        throw new Error('Error uploading image to Cloudinary');
    }
};

export default uploadImage;
```
### 3. Ініціалізація авторизації на клієнті (InitAuth.jsx)
```javascript
import { useEffect, useRef } from "react";
import { useDispatch, useSelector } from "react-redux";
import { checkAuth } from "../store/auth/authSlice";
import LoadingScreen from "../components/LoadingScreen";

const InitAuth = ({ children }) => {
  const dispatch = useDispatch();
  const { loading } = useSelector((state) => state.auth);
  const didRun = useRef(false);

  useEffect(() => {
    if (!didRun.current) {
      dispatch(checkAuth());
      didRun.current = true;
    }
  }, [dispatch]);

  if (loading && !didRun.current) return <LoadingScreen />;

  return children;
};

export default InitAuth;
```
### 4. Список аукціонів на клієнті (AuctionList.jsx)
```javascript
import { useState } from "react";
import AuctionCard from "../components/AuctionCard";
import { useQuery } from "@tanstack/react-query";
import { getAuctions } from "../api/auction";
import LoadingScreen from "../components/LoadingScreen";

export const AuctionList = () => {
  const [filter, setFilter] = useState("all");
  const { data, isLoading } = useQuery({
    queryKey: ["allAuction"],
    queryFn: getAuctions,
    staleTime: 30 * 1000,
  });

  if (isLoading) return <LoadingScreen />;

  const categories = ["all", ...new Set(data?.map((auction) => auction.itemCategory))];
  const filteredAuctions = filter === "all" ? data : data?.filter((auction) => auction.itemCategory === filter);

  return (
      <div className="min-h-screen bg-[#FFFFFF] text-[#0A0A0A]">
        <main className="max-w-7xl mx-auto px-6 py-12">
          <div className="mb-12 backdrop-blur-md bg-white/30 rounded-xl p-6 flex flex-wrap gap-3 shadow-md">
            <h2 className="w-full text-2xl md:text-3xl font-serif font-semibold mb-4">Категорії</h2>
            {categories.map((category) => (
                <button
                    key={category}
                    onClick={() => setFilter(category)}
                    className={`px-5 py-2 rounded-lg text-sm font-semibold transition-all duration-300 hover:scale-105 ${
                        filter === category ? "bg-[#8B5CF6] text-white shadow-lg" : "bg-[#F5F0E8] text-[#0A0A0A] hover:bg-[#FFE5D9]"
                    }`}
                >
                  {category.charAt(0).toUpperCase() + category.slice(1)}
                </button>
            ))}
          </div>

          <div className="mb-10">
            <h2 className="text-4xl md:text-5xl font-sans font-extrabold mb-2">
              {filter === "all" ? "Всі аукціони" : `${filter} Аукціони`}
            </h2>
            <span className="text-lg font-serif text-gray-500">
            ({filteredAuctions.length} предметів)
          </span>
          </div>

          {filteredAuctions.length === 0 ? (
              <div className="text-center py-20">
                <p className="text-lg font-serif text-gray-400">Аукціони в цій категорії відсутні.</p>
              </div>
          ) : (
              <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 3xl:grid-cols-4 gap-8 auto-rows-fr">
                {filteredAuctions.map((auction) => (
                    <div key={auction._id} className="transition-transform duration-300 hover:-translate-y-1">
                      <AuctionCard auction={auction} />
                    </div>
                ))}
              </div>
          )}
        </main>
      </div>
  );
};
```
### 5. Підключення до бази даних (connection.js)
```javascript
import mongoose from "mongoose";
import dotenv from "dotenv";
dotenv.config();

export const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGO_URL)
        console.log('Connected to MongoDB');
    } catch (error) {
        console.log('Error connecting to MongoDB')
    }
}
```
### 6. Налаштування завантаження файлів (multer.js)
```javascript
import multer from 'multer';
import { CloudinaryStorage } from 'multer-storage-cloudinary';
import { v2 as cloudinary } from 'cloudinary';

const storage = new CloudinaryStorage({
    cloudinary: cloudinary,
    params: {
        folder: 'uploads',
        format: async (req, file) => 'png',
        public_id: (req, file) => `${Date.now()}-${file.originalname}`,
    },
});

const upload = multer({ storage });

export default upload;
```
## Висновки
Під час виконання лабораторної роботи були опановані та реалізовані такі навички:

- Створення повноцінного frontend інтерфейсу з React + TypeScript.  
- Інтеграція з backend API та робота з JWT аутентифікацією.  
- Використання React Hook Form та Zod для валідації форм.  
- Реалізація CRUD операцій із глобальним управлінням станом через Zustand.  
- Покращення UX через toast сповіщення, skeleton loaders та responsive дизайн.  
- Розуміння паттернів Protected Routes та Container/Presentational компонентів.  

В результаті сформовано застосунок, який дозволяє ефективно управляти сутностями, забезпечує захист даних та зручний інтерфейс для користувача.

---

## Відповіді на контрольні запитання

1. **Контрольовані vs неконтрольовані компоненти**  
   Контрольовані компоненти отримують значення та зміну через props та стан, тоді як неконтрольовані використовують референси DOM (`ref`) для отримання значень. React Hook Form використовує неконтрольовані компоненти для оптимізації продуктивності та мінімізації перерендерів.

2. **JWT аутентифікація**  
   Після логіну сервер повертає токен, який зберігається у localStorage або sessionStorage. При кожному запиті токен додається до заголовків. Альтернатива — HttpOnly cookie.

3. **Zod**  
   TypeScript-first бібліотека для валідації та парсингу даних. Переваги: автоматичне виведення типів, детальні повідомлення про помилки, простота інтеграції з React Hook Form.

4. **Protected Routes**  
   Це маршрути, доступ до яких має лише аутентифікований користувач. Реалізується через компонент-обгортку, що перевіряє стан аутентифікації та робить redirect на login при відсутності токена.

5. **Context API vs Zustand**  
   Context API підходить для невеликих глобальних даних, як тема або мова, але викликає зайві ре-рендери. Zustand краще для середніх і великих застосунків із частими оновленнями стану завдяки селекторам та легкій оптимізації.

6. **Optimistic updates**  
   Миттєве оновлення UI перед підтвердженням від сервера для кращого UX. Використовується у CRUD операціях, коли затримка відповіді може сповільнити інтерфейс.

7. **Централізована обробка помилок**  
   Можна створити глобальний error handler, який перехоплює Axios помилки, логічно обробляє їх та показує toast повідомлення.

8. **Container/Presentational компонентів**  
   Container відповідає за логіку та стан, Presentational — за UI. Переваги: чисті компоненти, легше тестувати та підтримувати.

9. **Skeleton loaders**  
   Це компоненти-заглушки під час завантаження даних. Покращують сприйняття швидкості та UX, зменшують "стрибки" контенту.

10. **Синхронізація локального стану з сервером**  
    Використовуються виклики API після CRUD операцій для оновлення стану у store (наприклад, через `fetchUsers()` після створення нового користувача) та toast сповіщення для підтвердження дії.

---
