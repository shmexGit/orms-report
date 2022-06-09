---
theme: dualboot
layout: intro
---

# NodeJS ORM (Typeorm/Mikro-orm/Prisma/Sequelize/Objection)

<div class="orms-logo">
  <img src="/logo/typeorm_logo.png" class="m-5 w-20 h-20"/>
  <img src="/logo/microorm_logo.png" class="m-5 w-20 h-20"/>
  <img src="/logo/prisma_logo.png" class="m-5 w-20 h-20"/>
  <img src="/logo/sequelize_logo.png" class="m-5 w-20 h-20"/>
  <img src="/logo/obj_logo.svg" class="m-5 w-20 h-20"/>
</div>

<div>
  <a href="https://github.com/shmexGit/orms-report" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---

# Что такое ORM?

<div v-click>
ORM (Object-Relational Mapping, объектно-реляционное отображение) — технология программирования, суть которой заключается в создании «виртуальной объектной базы данных».
</div>

```ts {all|3|7-20|all}
// Что необходимо написать без ORM
import { createPool, Pool} from 'mysql';
let pool: Pool;
/*
  Инициализация pool
*/
export const execute = <T>(query: string, params: string[] | Object): Promise<T> => {
  try {
    if (!pool) throw new Error('Pool was not created. Ensure pool is created when running the app.');
    return new Promise<T>((resolve, reject) => {
      pool.query(query, params, (error, results) => {
        if (error) reject(error);
        else resolve(results);
      });
    });
  } catch (error) {
    console.error('[mysql.connector][execute][Error]: ', error);
    throw new Error('failed to execute MySQL query');
  }
}

```

---

# Плюсы и минусы ORM

Плюсы:

<v-clicks>

- ORM облегчают реализацию модели предметной области;
- ORM помогают уменьшить объем кода;
- ORM требуют, чтобы вы практически не писали SQL;
- Многие ORM абстрагируются от деталей, специфичных для базы данных.

</v-clicks>

Минусы:

<v-clicks>

- object–relational impedance mismatch;
- синхронизация;
- ORM, как правило, имеют большой API из-за сложности, которую они инкапсулируют;
- необходимость SQL.

</v-clicks>

---

# Основные паттерны используемые в ORM

Паттерны архитектуры источников данных:

<v-clicks>

- Active Record
- Data Mapper

</v-clicks>

Паттерны Объектно-Реляционной логики

<v-clicks>

- Identity Map
- Unit of Work

</v-clicks>

Паттерны обработки Объектно-Реляционных метаданных

<v-clicks>

- Repository

</v-clicks>

<div class="pattern-imgs">
  <img src="/patterns/activ-record.png"/>
  <img src="/patterns/data-mapper.png"/>
  <img src="/patterns/identity-map.png"/>
  <img src="/patterns/unit-of-work.png" class="w-50 h-40"/>
</div>
<img src="/patterns/repository.png" class="m-10 w-100 h-50"/>

---

# Миграции

<v-clicks>

Миграция — это набор шагов для перевода схемы базы данных из одного состояния в другое. Первая миграция обычно создает таблицы и индексы. Последующие миграции могут добавлять или удалять столбцы, вводить новые индексы или создавать новые таблицы. В зависимости от инструмента миграции, миграция может осуществляться в форме операторов SQL или программного кода, который будет преобразован в операторы SQL (как в случае с ActiveRecord и SQLAlchemy).

</v-clicks>

## Практики

<v-clicks>

- ошибки;
- не пытайтесь оптимизировать время выполнения;
- подтвердите вашу миграцию;
- план отката;
- пишите много логов;
- cине-зеленое развертывание;
- используйте транзакцию SQL для каждого ресурса;
- запускайте партиями;

</v-clicks>

---

# Typeorm

**TypeORM** — это ORM, который может работать на платформах NodeJS, Browser, Cordova, PhoneGap, Ionic, React Native, NativeScript, Expo и Electron и может использоваться с TypeScript и JavaScript (ES5, ES6, ES7, ES8). Его цель — всегда поддерживать новейшие функции JavaScript и предоставлять дополнительные функции, помогающие разрабатывать любые приложения, использующие базы данных — от небольших приложений с несколькими таблицами до крупномасштабных корпоративных приложений с несколькими базами данных.

## Подерживаемые базы данных
- mongodb
- postgres
- mysql
- sqlite
- mssql

<br>

```bash
yarn add typeorm pg reflect-metadata
yarn add -D @type/node
```

---

# Typeorm - сущности

**Entity** — это класс, который сопоставляется с таблицей базы данных (или коллекцией при использовании MongoDB). Вы можете создать объект, определив новый класс и пометив его с помощью декоратора @Entity():

```ts {all|9-10|11-12|13-15|16-17|18-19|all}
import { Entity, PrimaryGeneratedColumn, Column, Generated, UpdateDateColumn,  } from "typeorm";
export enum UserRole {
  ADMIN = "admin",
  EDITOR = "editor",
  GHOST = "ghost",
};
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  @Column({ type: "enum", enum: UserRole, default: UserRole.GHOST })
  role: UserRole;
  @Column()
  @Generated("uuid")
  uuid: string;
  @UpdateDateColumn({ type: "timestamp", default: () => 'CURRENT_TIMESTAMP(6)', onUpdate: 'CURRENT_TIMESTAMP(6)' })
  updatedAt: Date;
  @CreateDateColumn({ type: "timestamp", default: () => 'CURRENT_TIMESTAMP(6)' })
  createdAt: Date;
}
```

---

# Typeorm - паттерны

В TypeORM вы можете использовать шаблоны Active Record и Data Mapper.
```ts {all|2-13|14|15-21|all}
import { BaseEntity, Entity, PrimaryGeneratedColumn } from "typeorm";
// Active Record
@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;
  static findByName(firstName: string, lastName: string) {
    return this.createQueryBuilder("user")
      .where("user.firstName = :firstName", { firstName })
      .andWhere("user.lastName = :lastName", { lastName })
      .getMany();
  }
}
const timber = await User.findByName("Timber", "Saw");
// Data Mapper
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
}
```
---

# Typeorm - Entity Manager и Repository

С помощью EntityManager вы можете управлять (вставлять, обновлять, удалять, загружать и т. д.) любой сущностью. EntityManager подобен набору всех репозиториев сущностей в одном месте.

```ts
const user = await entityManager.findOneBy(User, {
    id: 1,
})
user.name = "Umed"
await entityManager.save(user)
```

Репозиторий похож на EntityManager, но его операции ограничены конкретным объектом. Вы можете получить доступ к репозиторию через EntityManager.

```ts
const userRepository = entityManager.getRepository(User)
const user = await userRepository.findOneBy({
    id: 1,
})
user.name = "Umed"
await userRepository.save(user)
```

---

# Typeorm - отношения

Отношения помогают вам легко работать со связанными сущностями. Существует несколько видов отношений:
- one-to-one - **@OneToOne**
- many-to-one - **@ManyToOne**
- one-to-many - **@OneToMany**
- many-to-many - **@ManyToMany**

---

**many-to-many** — это отношение, в котором A содержит несколько экземпляров B, а B содержит несколько экземпляров A. Возьмем, например, объекты Вопрос и Категория. У вопроса может быть несколько категорий, и в каждой категории может быть несколько вопросов.


```ts {all|1-8|10-21|all}
@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}

@Entity()
export class Question {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToMany(() => Category)
  @JoinTable()
  categories: Category[];
}
```

---

**@JoinTable** используется для отношений «многие ко многим» и описывает столбцы соединения «соединяемой» таблицы. Соединительная таблица — это специальная отдельная таблица, автоматически создаваемая TypeORM, со столбцами, которые относятся к связанным сущностям. Вы можете изменить имена столбцов в соединительных таблицах и столбцах, на которые они ссылаются, с помощью @JoinColumn: вы также можете изменить имя сгенерированной «соединительной» таблицы. Если целевая таблица имеет составные первичные ключи, то массив свойств должен быть задан в @JoinTable.

**@JoinColumn** - определяет какая сторона отношения содержит столбец соединения с внешним ключом и позволяет настроить имя столбца соединения, а также имя столбца, на который указывает ссылка.

Результат:
- Таблица `category` с полями `id, name`
- Таблица `question` с полями `id, title`
- Таблица `question_categories_category` с полями `questionId, categoryId`

---

**many-to-one / one-to-many** - это отношение, в котором A содержит несколько экземпляров B, а B содержит только один экземпляр A. Возьмем, например, объекты *User* и *Photo*. У пользователя может быть несколько фотографий, но каждая фотография принадлежит только одному пользователю.

```ts {all|10-11|all}
@Entity()
export class Photo {

  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  @ManyToOne(() => User, user => user.photos)
  user: User;

}
```

---

```ts
@Entity()
export class User {

  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Photo, photo => photo.user)
  photos: Photo[];

}
```

**@OneToMany** не может существовать без **@ManyToOne**. Однако обратное не требуется: если вас интересует только отношение **@ManyToOne**, вы можете определить его, не используя **@OneToMany** для связанного объекта.

---

# Typeorm - опции поиска

```ts
const users = await userRepository.find({
  relations: {
    profile: true,
    photos: true,
    videos: true,
  },
})
```

```sql
SELECT * FROM "user"
LEFT JOIN "profile" ON "profile"."id" = "user"."profileId"
LEFT JOIN "photos" ON "photos"."id" = "user"."photoId"
LEFT JOIN "videos" ON "videos"."id" = "user"."videoId"
```

```ts
const users = await userRepository.find({
  where: {
    firstName: "Timber",
    lastName: "Saw",
  },
})
```

```sql
SELECT * FROM "user"
WHERE "firstName" = 'Timber' AND "lastName" = 'Saw'
```

---

```ts
const users = await userRepository.find({
  relations: {
    project: true,
  },
  where: {
    project: {
      name: "TypeORM",
      initials: "TORM",
    },
  },
})
```
```sql
SELECT * FROM "user"
LEFT JOIN "project" ON "project"."id" = "user"."projectId"
WHERE "project"."name" = 'TypeORM' AND "project"."initials" = 'TORM'
```

```ts
const users = await userRepository.find({
  where: [
    { firstName: "Timber", lastName: "Saw" },
    { firstName: "Stan", lastName: "Lee" },
  ],
});
```

```sql
SELECT * FROM "user" WHERE ("firstName" = 'Timber' AND "lastName" = 'Saw') OR ("firstName" = 'Stan' AND "lastName" = 'Lee')
```

---

```ts
const users = await userRepository.find({
  select: ["name", "id"],
  order: {
    name: "ASC",
    id: "DESC",
  },
  skip: 5,
  take: 10,
});
```

```sql
SELECT "name", "id" FROM "user" ORDER BY "name" ASC, "id" DESC LIMIT 10 OFFSET 5
```

```ts
import { Like } from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
  title: Like("%out #%"),
});
```

```sql
SELECT * FROM "post" WHERE "title" LIKE '%out #%'
```

---

# Typeorm - транзакции

```ts
import { getManager } from "typeorm";

await getManager().transaction(async transactionalEntityManager => {
  await transactionalEntityManager.save(users);
  await transactionalEntityManager.save(photos);
  // ...
});
```

Следующие драйверы баз данных поддерживают стандартные уровни изоляции (READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE): 
- MySQL 
- Postgres
- MSSQL

```ts
import {getManager} from "typeorm";

await getManager().transaction("SERIALIZABLE", transactionalEntityManager => {
//...    
});

```

---

# Typeorm - CLI

```bash
typeorm entity:create -n User
typeorm migration:generate -n UserMigration
typeorm migration:run
typeorm migration:revert
typeorm migration:show
```

---

# Mikro-orm

ORM TypeScript для Node.js на основе шаблонов Data Mapper, Unit of Work и Identity Map.

```bash
yarn add @mikro-orm/core @mikro-orm/mongodb     # for mongo
yarn add @mikro-orm/core @mikro-orm/mysql       # for mysql/mariadb
yarn add @mikro-orm/core @mikro-orm/mariadb     # for mysql/mariadb
yarn add @mikro-orm/core @mikro-orm/postgresql  # for postgresql
yarn add @mikro-orm/core @mikro-orm/sqlite      # for sqlite
```

---

# Mikro-orm - сущности

```ts {all|3-4|5-8|9-10|11-12|13-14|15-16|all}
@Entity()
export class Author {
  @PrimaryKey()
  _id: ObjectID;
  @Property()
  @Unique()
  @Index({ name: 'born_index' })  // или с автоматической генерацией @Index()
  bar!: string;
  @Property({ defaultRaw: 'now' })
  baz!: Date;
  @Property()
  createdAt: Date = new Date();
  @Property({ onUpdate: () => new Date() })
  updatedAt: Date = new Date();
  @Enum(() => UserRole)
  role: UserRole;
}
export enum UserRole {
  ADMIN = 'admin',
  MODERATOR = 'moderator',
  USER = 'user',
}
```


---

# Mikro-orm - паттерны

**MikroORM** использует **Identity Map** в фоновом режиме для отслеживания объектов. Это означает, что всякий раз, когда вы извлекаете объект через **EntityManager**, **MikroORM** будет хранить ссылку на него внутри своего **UnitOfWork** и всегда будет возвращать один и тот же его экземпляр, даже если вы запрашиваете один объект через разные свойства. Это также означает, что вы можете сравнивать сущности с помощью операторов строгого равенства (=== и !==):

```ts
const authorRepository = orm.em.getRepository(Author);
const jon = await authorRepository.findOne({ name: 'Jon Snow' }, ['books']);
const jon2 = await authorRepository.findOne({ email: 'snow@wall.st' });
const authors = await authorRepository.findAll(['books']);

// identity map in action
console.log(jon === authors[0]); // true
console.log(jon === jon2); // true

// as we always have one instance, books will be populated also here
console.log(jon2.books);
```

---

# Mikro-orm - Entity Manager и Repository

Есть два метода, которые мы должны сначала описать, чтобы понять, как работает сохранение в MikroORM: 
- em.persist() - используется для пометки новых объектов для сохранения в будущем;
- em.flush() - чтобы понять *flush*, давайте сначала определим, что такое управляемый объект: объект является управляемым, если он получен из базы данных (через *em.find()*, *em.findOne()*) или зарегистрирован как новый через *em.persist()*. *em.flush()* будет проходить через все управляемые объекты, вычислять соответствующие наборы изменений и выполнять соответствующие запросы к базе данных.

**Репозитории сущностей** — это тонкие слои поверх *EntityManager*. Они действуют как точка расширения, поэтому мы можем добавлять собственные методы или даже изменять существующие. Реализация *EntityRepository* по умолчанию просто перенаправляет вызовы базовому экземпляру *EntityManager*.

> Класс EntityRepository содержит тип объекта, поэтому нам не нужно передавать его при каждом вызове find или findOne.

<br>

```ts
const booksRepository = em.getRepository(Book);
const books = await booksRepository.find({ author: '...' }, { 
  populate: ['author'],
  limit: 1,
  offset: 2,
  orderBy: { title: QueryOrder.DESC },
});
console.log(books); // Book[]
```

---

# Mikro-orm - отношения

**Collections**

Свойства OneToMany и ManyToMany хранятся в оболочке Collection.

**OneToMany**

```ts {all|5-6|12-13|14-16|all}
@Entity()
export class Book {
  @PrimaryKey()
  _id!: ObjectId;
  @ManyToOne()
  author!: Author;
}
@Entity()
export class Author {
  @PrimaryKey()
  _id!: ObjectId;
  @OneToMany(() => Book, book => book.author)
  books1 = new Collection<Book>(this);
  // альтернатива
  @OneToMany({ entity: () => Book, mappedBy: 'author' })
  books2 = new Collection<Book>(this);
}
```

---

**ManyToMany**

Для ManyToMany драйверы SQL используют сводную таблицу, которая содержит ссылки на обе сущности.

```ts {all|1-3|5-10|all}
// Одностороний
@ManyToMany(() => Book)
books = new Collection<Book>(this);

// Двухстороний
@ManyToMany(() => BookTag, tag => tag.books, { owner: true })
tags = new Collection<BookTag>(this);

@ManyToMany(() => Book, book => book.tags)
books = new Collection<Book>(this);
```

---

# Mikro-orm - опции поиска

```ts {all|1|2-3|4-5|6-7|8|9|10|all}
const users = await em.find(User, { firstName: 'John' });
const id = 1;
const users = await em.find(User, { organization: id });
const ref = await em.getReference(Organization, id);
const users = await em.find(User, { organization: ref });
const ent = await em.findOne(Organization, id);
const users = await em.find(User, { organization: ent });
const users = await em.find(User, { $and: [{ id: { $nin: [3, 4] } }, { id: { $gt: 2 } }] });
const users = await em.find(User, [1, 2, 3, 4, 5]);
const user1 = await em.findOne(User, 1);
```

Как видно из пятого примера, можно также использовать такие операторы, как $and, $or, $gte, $gt, $lte, $lt, $in, $nin, $eq, $ne, $like, $re. 

---

# Mikro-orm - транзакции

По большей части, MikroORM уже позаботится о правильном разграничении транзакций за вас: все операции записи (INSERT/UPDATE/DELETE) ставятся в очередь до тех пор, пока не будет вызвана em.flush(), которая оборачивает все эти изменения в одну транзакцию.

Тем не менее, MikroORM также позволяет (и поощряет) вам самостоятельно контролировать транзакции.

Первый подход заключается в использовании неявной обработки транзакций, предоставляемой MikroORM EntityManager. Учитывая следующий фрагмент кода без явного разграничения транзакций:


```ts
const user = new User(...);
user.name = 'George';
await orm.em.persistAndFlush(user);
```

Поскольку в приведенном выше коде мы не делаем никакого пользовательского разграничения транзакций, *em.flush()* запустится и зафиксирует/откатит транзакцию. Такое поведение стало возможным благодаря агрегированию операций *DML* с помощью *MikroORM* и является достаточным, если все манипуляции с данными, являющиеся частью единицы работы, происходят через модель предметной области и, следовательно, *ORM*.

---

Явной альтернативой является использование API транзакций напрямую для контроля границ. Тогда код выглядит так:

```ts
await orm.em.transactional(em => {
  //... do some work
  const user = new User(...);
  user.name = 'George';
  em.persist(user);
});
```

Или вы можете явно использовать методы *begin/commit/rollback*. Следующий пример эквивалентен предыдущему:

```ts
const em = orm.em.fork();
await em.begin();

try {
  //... do some work
  const user = new User(...);
  user.name = 'George';
  em.persist(user);
  await em.commit(); // will flush before making the actual commit query
} catch (e) {
  await em.rollback();
  throw e;
}
```

---

Явное разграничение транзакций требуется, когда вы хотите включить пользовательские операции *DBAL* в единицу работы или когда вы хотите использовать некоторые методы *API EntityManager*, для которых требуется активная транзакция. Такие методы будут вызывать *ValidationError*, чтобы сообщить вам об этом требовании. 

em.transactional(cb) очистит внутренний EntityManager перед фиксацией транзакции.

---

# Mikro-orm - CLI

```bash
  mikro-orm migration:create      # Создать новую миграцию с текущей схемой разница
  mikro-orm migration:up          # Переход на последнюю версию
  mikro-orm migration:down        # Перейти на один шаг вниз
  mikro-orm migration:list        # Список всех выполненных миграций
  mikro-orm migration:pending     # Список всех ожидающих миграций
```

---

# Prisma

Поддерживаемые базы данных:

- MySQL
- PostgreSQL
- SQLite
- MSSQL (в разработке)
- MongoDB Connector (в разработке)

Предоставляемые инструменты:

- Prisma Client: автоматически генерируемый и типобезопасный клиент для БД;
- Prisma Migrate: декларативное моделирование данных и настраиваемые миграции;
- Prisma Studio: современный пользовательский интерфейс для просмотра и редактирования данных;
- Prisma VSCode Extension: расширение для VSCode, обеспечивающее подсветку синтаксиса, автозавершение, быстрые исправления и др.

---

# Prisma - схема

Каждый проект, в котором используется инструмент из набора инструментов Prisma, начинается с файла схемы Prisma. Схема Prisma позволяет разработчикам определять свои модели приложений на интуитивно понятном языке моделирования данных. Он также содержит подключение к базе данных и определяет генератор.

```bash {all|1-4|6-8|10-15|all}
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}
```

---

Каждая модель привязана к таблице в БД и является основой для генерируемого Prisma Client интерфейса доступа к данным.

В приведенной схеме мы настраиваем следующее:

- Источник данных (data source): определяет соединение с БД (через переменную среды окружения);
- Генератор (generator): сообщает, что мы хотим сгенерировать Prisma Client;
- Модель данных (data model): определяет модели приложения.

Главными задачами моделей является следующее:

- представление таблицы в БД
- предоставление основы для запросов Prisma Client

Для использования Prisma Client прежде всего необходимо установить соответствующий пакет из npm:

```bash
yarn add @prisma/client
```

---

# Prisma - паттерны

Чтобы понять, чем реализация шаблона Data Mapper в Prisma концептуально отличается от традиционных ORM Data Mapper, вот краткое сравнение их концепций и строительных блоков:
<img src="/prisma-pattern.png" class="w-150 h-80"/>


---

# Prisma - отношения

OneToMany

```bash {all|8}
model User {
  id    Int    @id @default(autoincrement())
  posts Post[]
}

model Post {
  id       Int  @id @default(autoincrement())
  author   User @relation(fields: [authorId], references: [id])
  authorId Int
}
```

```sql
CREATE TABLE "User" (
    id SERIAL PRIMARY KEY
);
CREATE TABLE "Post" (
    id SERIAL PRIMARY KEY,
    "authorId" integer NOT NULL,
    FOREIGN KEY ("authorId") REFERENCES "User"(id)
);
```

---

ManyToMany

```bash {all|14-15|16-17|all}
model Post {
  id         Int    @id @default(autoincrement())
  title      String
  categories CategoriesOnPosts[]
}

model Category {
  id    Int  @id @default(autoincrement())
  name  String
  posts CategoriesOnPosts[]
}

model CategoriesOnPosts {
  post       Post     @relation(fields: [postId], references: [id])
  postId     Int
  category   Category @relation(fields: [categoryId], references: [id])
  categoryId Int
  assignedAt DateTime @default(now())
  assignedBy String

  @@id([postId, categoryId])
}
```

---

# Prisma - опции поиска

```ts {all|6-8|15-16|17-21|all}
// include
const getUser = await prisma.user.findUnique({
  where: {
    id: 19,
  },
  include: {
    posts: true,
  },
})
// select
const getUser = await prisma.user.findUnique({
  where: {
    id: 19,
  },
  select: {
    name: true,
    posts: {
      select: {
        title: true,
      },
    },
  },
})
```

---



---

# Prisma - транзакции

---

# Prisma - CLI

```bash
prisma init     # Настройка Prisma для вашего приложения 
prisma generate # Генерация артефактов (например, клиент Prisma) 
prisma db       # Управляйте схемой и жизненным циклом вашей базы данных миграция Перенесите вашу базу данных
prisma migrate dev --name init # Создать миграцию
```
