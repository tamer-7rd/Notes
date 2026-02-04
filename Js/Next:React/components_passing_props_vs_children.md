# React: Передача компонентов - Props vs Children

## 📦 Два способа передачи React-элементов

### 1. Через Props (Явная передача)

```tsx
type ButtonProps = {
  icon: React.ReactNode
  label: string
}

function Button({ icon, label }: ButtonProps) {
  return (
    <button>
      {icon}
      <span>{label}</span>
    </button>
  )
}

// Использование
<Button icon={<StarIcon />} label="Favorite" />
```

### 2. Через Children (Неявная передача)

```tsx
type ButtonProps = {
  children: React.ReactNode
}

function Button({ children }: ButtonProps) {
  return <button>{children}</button>
}

// Использование
<Button>
  <StarIcon />
  <span>Favorite</span>
</Button>
```

## 🎯 Когда использовать Props

✅ **Несколько слотов** с конкретным назначением
```tsx
<Card 
  header={<Title />}
  content={<Text />}
  footer={<Actions />}
/>
```

✅ **Опциональные элементы** с понятными именами
```tsx
<Alert 
  icon={<WarningIcon />}
  action={<Button />}
/>
```

✅ **Строгий контракт** - ясно, что куда идет
```tsx
<Avatar 
  image={<img />}
  badge={<StatusDot />}
/>
```

## 🎯 Когда использовать Children

✅ **Свободная композиция** - любое содержимое
```tsx
<Modal>
  <h1>Title</h1>
  <p>Content</p>
  <button>Close</button>
</Modal>
```

✅ **Обертки и контейнеры**
```tsx
<Layout>
  <Sidebar />
  <MainContent />
</Layout>
```

✅ **Один основной слот** контента
```tsx
<Card>
  <p>Anything here</p>
</Card>
```

## 💡 Комбинированный подход

Можно использовать **и props, и children** одновременно:

```tsx
type CardProps = {
  header: React.ReactNode
  footer?: React.ReactNode
  children: React.ReactNode
}

function Card({ header, footer, children }: CardProps) {
  return (
    <div className="card">
      <div className="card-header">{header}</div>
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  )
}

// Использование
<Card 
  header={<h2>Title</h2>}
  footer={<Button>OK</Button>}
>
  <p>Main content goes here</p>
</Card>
```

## 🔑 Правило выбора

| Критерий | Props | Children |
|----------|-------|----------|
| Количество слотов | Несколько | Один |
| Гибкость | Строгая структура | Свободная композиция |
| Читаемость | Явная | Естественная (HTML-like) |
| API ясность | Высокая | Средняя |

## 📌 React.ReactNode vs JSX.Element

```tsx
// React.ReactNode - максимально широкий тип
// Принимает: JSX, string, number, null, undefined, array
children: React.ReactNode

// JSX.Element - только React-элементы
// Принимает: только JSX (<Component />)
icon: JSX.Element

// Чаще используют React.ReactNode для гибкости
```

## 🎓 Best Practices

1. **Props** для специфичного контента (icon, badge, avatar)
2. **Children** для основного содержимого (текст, вложенные компоненты)
3. Комбинируй оба подхода для сложных компонентов
4. Именуй props понятно: `leftIcon`, `rightIcon` вместо `icon1`, `icon2`