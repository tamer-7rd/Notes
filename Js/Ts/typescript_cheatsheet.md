
# 📝 Шпаргалка по TypeScript (с нуля до рабочего кода)

---

## 1. Базовые типы
```ts
let title: string = "Новости";   // строка
let views: number = 123;         // число
let isPublished: boolean = true; // логическое
let nothing: null = null;        // пустое
let notSet: undefined = undefined; // ещё не задано
```

---

## 2. Массивы
```ts
let tags: string[] = ["AI", "Next.js"];  // массив строк
let numbers: number[] = [1, 2, 3];       // массив чисел
```

---

## 3. Объекты
```ts
type Article = {
  id: string;
  title: string;
  summary?: string; // ? значит поле может быть, а может не быть
};

let article: Article = {
  id: "1",
  title: "Первая статья",
};
```

---

## 4. Union-типы (несколько вариантов)
```ts
type Status = "idle" | "loading" | "success" | "error";

let status: Status = "loading"; // ок
status = "error";               // ок
status = "abc";                 // ❌ ошибка
```

---

## 5. Функции
```ts
function add(a: number, b: number): number {
  return a + b;
}
```

- `a: number, b: number` → входные параметры должны быть числами  
- `: number` после скобок → функция возвращает число  

---

## 6. async/await + Promise
```ts
async function fetchArticles(): Promise<Article[]> {
  const res = await fetch("https://api.example.com/articles");
  return res.json() as Promise<Article[]>;
}
```
- `Promise<Article[]>` → функция вернёт обещание с массивом статей.  

---

## 7. Generics (универсальные функции)
```ts
function take<T>(arr: T[], count: number): T[] {
  return arr.slice(0, count);
}

take([1, 2, 3], 2);           // T = number
take(["a", "b", "c"], 2);     // T = string
```
- `<T>` → переменная для типа (подставляется автоматически).  
- `T[]` → массив элементов этого типа.  

---

## 8. Props в React
```tsx
type ArticleCardProps = {
  title: string;
  summary?: string;
};

function ArticleCard({ title, summary }: ArticleCardProps) {
  return (
    <article>
      <h2>{title}</h2>
      <p>{summary}</p>
    </article>
  );
}

// использование:
<ArticleCard title="AI news" summary="ИИ меняет мир" />
```

---

## 9. Типизация API-ответов
```ts
// DTO (как сервер отдаёт)
type ArticleDTO = {
  id: string;
  title: string;
  published_at: string;
};

// Domain (как удобно в коде)
type Article = {
  id: string;
  title: string;
  publishedAt: Date;
};

// Конвертация
function toArticle(dto: ArticleDTO): Article {
  return {
    id: dto.id,
    title: dto.title,
    publishedAt: new Date(dto.published_at),
  };
}
```

---

✅ Суперкратко:  
- `string[]` — массив строк  
- `?` — опциональное поле  
- `|` — несколько вариантов (union)  
- `: Type` — указывает тип переменной, параметра или результата  
- `<T>` — универсальный тип (дженерик)  
