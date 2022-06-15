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

# Seeding

```ts {all|6|7-10|12-14|all}
import { MigrationInterface, QueryRunner } from "typeorm";

import { User } from "src/entities/user.entity";
import { userSeed } from "src/seeders/user.seed";

export class UserSeedTIMESTAMP implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    const userRepository = queryRunner.connection.getRepository(User);
    return userRepository.save(userSeed);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    return queryRunner.connection.getRepository(User).clear();
  }
}
```

---

# Typeorm

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
import { Entity, PrimaryGeneratedColumn, Column, Generated, UpdateDateColumn } from "typeorm";
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

```ts {all|1|all}
const user = await entityManager.findOneBy(User, {
    id: 1,
})
user.name = "Umed"
await entityManager.save(user)
```

Репозиторий похож на EntityManager, но его операции ограничены конкретным объектом. Вы можете получить доступ к репозиторию через EntityManager.

```ts {all|1|2|all}
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

## ManyToMany

Это отношение, в котором A содержит несколько экземпляров B, а B содержит несколько экземпляров A. Возьмем, например, объекты Вопрос и Категория. У вопроса может быть несколько категорий, и в каждой категории может быть несколько вопросов.

```ts {all|1-8|9-20|all}
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

## ManyToOne

Это отношение, в котором A содержит несколько экземпляров B, а B содержит только один экземпляр A. Возьмем, например, объекты *User* и *Photo*. У пользователя может быть несколько фотографий, но каждая фотография принадлежит только одному пользователю.

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

## OneToMany

```ts {all|10-11|all}
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

```ts {all|2-6|all}
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

```ts {all|2-5|all}
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

```ts {all|5-10|all}
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

```ts {all|2-5|all}
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

```ts {all|2|3-6|7|8|all}
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

```ts {all|4|all}
import { Like } from "typeorm"; // Различные условия для WHERE импортятся как функции из typeorm

const loadedPosts = await connection.getRepository(Post).find({
  title: Like("%out #%"),
});
```

```sql
SELECT * FROM "post" WHERE "title" LIKE '%out #%'
```

---

# Typeorm - транзакции

## Управляемые транзакции

```ts {all|0}
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

```ts {all|0}
import { getManager } from "typeorm";

await getManager().transaction("SERIALIZABLE", transactionalEntityManager => {
//...    
});

```

---

## Неуправляемые транзакции

```ts {all|3|4|5|6|7|8|13|15|all}
import { getConnection } from "typeorm";

const connection = getConnection();
const queryRunner = connection.createQueryRunner();
await queryRunner.connect();
await queryRunner.query("SELECT * FROM users");
const users = await queryRunner.manager.find(User);
await queryRunner.startTransaction();
try {
  await queryRunner.manager.save(user1);
  await queryRunner.manager.save(user2);
  await queryRunner.manager.save(photos);
  await queryRunner.commitTransaction();
} catch (err) {
  await queryRunner.rollbackTransaction();
} finally {
  await queryRunner.release();
}
```

---

# Typeorm - CLI

```bash {all|1|2|3|4|5|all}
 $ typeorm entity:create -n User
 $ typeorm migration:generate -n UserMigration
 $ typeorm migration:run
 $ typeorm migration:revert
 $ typeorm migration:show
```

---

# Mikro-orm

ORM TypeScript для Node.js на основе шаблонов Data Mapper, Unit of Work и Identity Map.

```bash
 $ yarn add @mikro-orm/core @mikro-orm/mongodb     # для mongo
 $ yarn add @mikro-orm/core @mikro-orm/mysql       # для mysql/mariadb
 $ yarn add @mikro-orm/core @mikro-orm/mariadb     # для mysql/mariadb
 $ yarn add @mikro-orm/core @mikro-orm/postgresql  # для postgresql
 $ yarn add @mikro-orm/core @mikro-orm/sqlite      # для sqlite
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

```ts
const authorRepository = orm.em.getRepository(Author);
const jon = await authorRepository.findOne({ name: 'Jon Snow' }, ['books']);
const jon2 = await authorRepository.findOne({ email: 'snow@wall.st' });
const authors = await authorRepository.findAll(['books']);

console.log(jon === authors[0]); // true
console.log(jon === jon2); // true
```

---

# Mikro-orm - Entity Manager и Repository

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

<br>

> Свойства OneToMany и ManyToMany хранятся в оболочке Collection. Для ManyToOne нет такой необходимости.

<br>

## OneToMany / ManyToOne

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

## ManyToMany

Для ManyToMany драйверы SQL используют сводную таблицу, которая содержит ссылки на обе сущности. Все ссылки хранятся в виде массива **ObjectIds** на объекте-владельце.

```ts {all|1-3|5-10|all}
// Одностороняя связь
@ManyToMany(() => Book)
books = new Collection<Book>(this);

// Двухстороняя связь
@ManyToMany(() => BookTag, tag => tag.books, { owner: true })
tags = new Collection<BookTag>(this);

@ManyToMany(() => Book, book => book.tags)
books = new Collection<Book>(this);
```

---

# Mikro-orm - опции поиска

```ts {all|1|2-3|4-5|6-7|8|9|10|11|all}
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
const tags = await em.findAll(BookTag, { populate: ['books.publisher.tests', 'books.author'] });
```

---

# Mikro-orm - транзакции

```ts {all|0}
const user = new User(...);
user.name = 'George';
await orm.em.persistAndFlush(user);
```

```ts {all|0}
await orm.em.transactional(em => {
  //... do some work
  const user = new User(...);
  user.name = 'George';
  em.persist(user);
});
```

```ts {all|0}
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

# Mikro-orm - CLI

```bash {all|1|2|3|4|5|6|7|8|9|10|all}
  $ mikro-orm migration:create --initial # Создать начальную миграцию (если готова схема)
  $ mikro-orm migration:up          # Переход на последнюю версию
  $ mikro-orm migration:down        # Откатить на один шаг назад
  $ mikro-orm migration:list        # Список всех выполненных миграций
  $ mikro-orm migration:pending     # Список всех ожидающих миграций
  $ mikro-orm migration:fresh       # Удалить базу и выполнить миграцию до последней версии
  $ mikro-orm seeder:create DatabaseSeeder  # генерация класса DatabaseSeeder
  $ mikro-orm seeder:create test            # генерация класса TestSeeder
  $ mikro-orm seeder:run                      # запуск вставки данных
  $ mikro-orm seeder:run --class=BookSeeder   # запуск определенного класса
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

**select** и **include** нельзя использовать на одном уровне:
```ts {all|5|8|all}
const getUser = await prisma.user.findUnique({
  where: {
    id: 19,
  },
  select: { // не работает
    email:  true
  }
  include: { // не работает
    posts: {
      select: {
        title: true
      }
    }
  },
})
```

---

```ts {all|9-14|all}
const result = await prisma.user.findMany({
  where: {
    email: {
      contains: 'prisma.io',
    },
  },
  select: {
    posts: {
      where: {
        published: false,
      },
      orderBy: {
        title: 'asc',
      },
      select: {
        title: true,
      },
    },
  },
})
```

---

### Операторы

```ts {all|4|7|9|10|13|all}
const result = await prisma.user.findMany({
  where: {
    name: {
      equals: 'Eleanor',
    },
    email: {
      not: 'elrm@mail.com',
    },
    id: { in: [22, 91, 14, 2, 5] },
    NOT: {
      name: { in: ['Saqui', 'Clementine', 'Bob'] },
    },
    id: { notIn: [22, 91, 14, 2, 5] },
  },
})
```

---

```ts {all|4|5|6|all}
const getPosts = await prisma.post.findMany({
  where: {
    likes: {
      lte: 9, // <= 9
      gt: 4,  // > 4
      gte: 8, // >= 8
    },
  },
})
```

Используйте полнотекстовый поиск для поиска в текстовом поле. Полнотекстовый поиск в настоящее время находится в предварительной версии и доступен только для *PostgreSQL* и *MySQL*. Чтобы использовать поиск, вам необходимо включить функцию предварительного просмотра **fullTextSearch**.

```sql {all|3|all}
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["fullTextSearch"]
}
```

---

```ts {all|4|all}
const result = await prisma.post.findMany({
  where: {
    title: {
      search: 'cat | dog',  // '!cat', 'cat & dog'
    },
  },
})
```

**mode** - поддерживается только коннекторами PostgreSQL и MongoDB.

Получить все записи Post, в которых title содержит prisma, без учета регистра.

```ts
const result = await prisma.post.findMany({
  where: {
    title: {
      contains: 'prisma',
      mode: 'insensitive',
    },
  },
})
```

---

AND и OR

```ts {all|3|15|all}
const result = await prisma.post.findMany({
  where: {
    OR: [
      {
        title: {
          contains: 'Prisma',
        },
      },
      {
        title: {
          contains: 'databases',
        },
      },
    ],
    AND: {
      published: false,
    },
  },
})
```

---

# Prisma - транзакции

## Вложеные записи

```ts {all|5-8|all}
const nestedWrite = await prisma.user.create({
  data: {
    email: 'imani@prisma.io',
    posts: {
      create: [
        { title: 'My first day at Prisma' },
        { title: 'How to configure a unique constraint in PostgreSQL' },
      ],
    },
  },
})
```
<br>

> Также стоит отметить, что операции выполняются в соответствии с порядком их размещения в транзакции.

<br>

## API $transaction

```ts {all|3|all}
const updateUsers = prisma.user.update(/**/);
const updateTeam = prisma.team.update(/**/);
await prisma.$transaction([updateUsers, updateTeam])
```

---

# Prisma - CLI

```bash
  $ prisma init     # Настройка Prisma для вашего приложения 
  $ prisma generate # Генерация артефактов (например, клиент Prisma) 
  $ prisma db       # Управляйте схемой и жизненным циклом вашей базы данных миграция Перенесите вашу базу данных
  $ prisma db pull  # Вытягивает схему из существующей базы данных, обновив схему Prisma
  $ prisma migrate dev # Создает миграцию
  $ prisma migrate dev --name init # Создает миграцию с именем
```

---

# Sequelize

Это ORM для работы с такими СУБД, как: 
- Postgres;
- MySQL;
- MariaDB;
- SQLite;
- MSSQL.

```bash
  $ yarn add sequelize pg pg-hstore # Postgres
```

---

# Sequelize - Repository mode

Данный функционал предоставляет **sequelize-typescript**.
Режим репозитория позволяет отделить статические операции, такие как поиск, создание и т. д., от определений модели. Это также расширяет возможности моделей.

```ts
const userRepository = sequelize.getRepository(User)
const addressRepository = sequelize.getRepository(Address)

userRepository.find({ include: [addressRepository] })
userRepository.create({ name: 'Bear' }, { include: [addressRepository] })
```

---

# Sequelize - модели

В качестве примера создадим модель User с полями firstName и lastName.

```ts
import { Sequelize, DataTypes } from 'sequelize';
const sequelize = new Sequelize('sqlite::memory:')

const User = sequelize.define(
  'User',
  {
    // Здесь определяются атрибуты модели
    firstName: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    lastName: {
      type: DataTypes.STRING,
      // allowNull по умолчанию имеет значение true
    },
  },
  {
    // Здесь определяются другие настройки модели
  }
);
// `sequelize.define` возвращает модель
console.log(User === sequelize.models.User) // true
```

---

Для использования декораторв необходимо поставить **sequelize-typescript**

```ts {all|3|4|5-8|17-19|all}
import {/**/} from 'sequelize-typescript';

@Table({ tableName: 'user_sequelize' })
export class UserSequelize extends Model {
  @PrimaryKey
  @AutoIncrement
  @Column
  id: number;

  @Column
  name: string;

  @IsNull
  @Column({ field: 'profile_image' })
  profileImage: string;

  @CreatedAt
  @Column({ field: 'created_at' })
  createdAt: Date;

  @UpdatedAt
  @Column({ field: 'updated_at' })
  updatedAt: Date;
}

```

---

# Sequelize - отношения

Отношения могут быть описаны непосредственно в модели с помощью декораторов *@HasMany*, *@HasOne*, *@BelongsTo*, *@BelongsToMany* и *@ForeignKey*.

## OneToMany

```ts {all|5-7|8-9|16-17|all}
@Table
class Player extends Model {
  @Column
  name: string
  @ForeignKey(() => Team)
  @Column
  teamId: number
  @BelongsTo(() => Team)
  team: Team
}

@Table
class Team extends Model {
  @Column
  name: string
  @HasMany(() => Player)
  players: Player[]
}

```

---

## ManyToMany

```ts{all|3-4|15-17|all}
@Table
class Book extends Model {
  @BelongsToMany(() => Author, () => BookAuthor)
  authors: Author[]
}

@Table
class Author extends Model {
  @BelongsToMany(() => Book, () => BookAuthor)
  books: Book[]
}

@Table
class BookAuthor extends Model {
  @ForeignKey(() => Book)
  @Column
  bookId: number

  @ForeignKey(() => Author)
  @Column
  authorId: number
}
```

---

# Sequelize - опции поиска

Чтобы использовать обширную базу операторв, необходи заимпортить *Op*

```ts {all|1|5-8|9|all}
import { Op } from 'sequelize';
Foo.findAll({
  where: {
    rank: {
      [Op.or]: {
        [Op.lt]: 1000,
        [Op.eq]: null
      }
    }, // rank < 1000 OR rank IS NULL
    {
      createdAt: {
        [Op.lt]: new Date(),
        [Op.gt]: new Date(new Date() - 24 * 60 * 60 * 1000)
      }
    }
  }
})
```

---

Использование функций в запросе:

```ts {all|4|all}
Post.findAll({
  where: {
    [Op.or]: [
      sequelize.where(sequelize.fn('char_length', sequelize.col('content')), 7),
      {
        content: {
          [Op.like]: 'Hello%',
        },
      },
    ],
  },
})
```

---

Сортировка и группировка:

```ts {all|2-19|20|21|22|all}
Submodel.findAll({
  order: [
    // Сортировка по заголовку (по убыванию)
    ['title', 'DESC'],

    // Сортировка по максимальному возврасту
    sequelize.fn('max', sequelize.col('age')),

    // Тоже самое, но по убыванию
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],

    // Сортировка по `createdAt` из связанной модели
    [Model, 'createdAt', 'DESC'],

    // Сортировка по `createdAt` из двух связанных моделей
    [Model, AnotherModel, 'createdAt', 'DESC'],

    // и т.д.
  ],
  group: 'name',
  offset: 5,
  limit: 10
})
```

---

# Sequelize - транзакции

Sequelize поддерживает два способа использования транзакций: 
- неуправляемые транзакции: фиксация и откат транзакции должны выполняться пользователем вручную (путем вызова соответствующих методов Sequelize). 
- управляемые транзакции: Sequelize автоматически откатывает транзакцию, если возникает какая-либо ошибка, или фиксирует транзакцию в противном случае.

## Неуправляемые транзакции

```ts {all|1|3-6|8|10|all}
const transaction = await sequelize.transaction();
try {
  const user = await User.create({
    firstName: 'Bart',
    lastName: 'Simpson'
  }, { transaction });
  // ...
  await t.commit();
} catch (error) {
  await t.rollback();
}
```

---

## Управляемые транзакции

Управляемые транзакции обрабатывают фиксацию или откат транзакции автоматически. Вы запускаете управляемую транзакцию, передавая обратный вызов в sequenceize.transaction. Этот обратный вызов может быть асинхронным (и обычно так и есть). 

```ts {all|2|3-6|8|all}
try {
  const result = await sequelize.transaction(async (transaction) => {
    const user = await User.create({
      firstName: 'Abraham',
      lastName: 'Lincoln'
    }, { transaction });
    //...
    return user;
  });
} catch (error) {
  // Уже произошел откат транзакции
}
```

---

# Sequelize - CLI

```bash {all|1-2|3-4|5-6|7-8|9-10|11-12|13-14|15-16|17-18|all}
# Генерация модели
$ sequelize-cli model:generate --name User --attributes firstName:string,lastName:string,email:string
# Запуск миграций
$ sequelize-cli db:migrate
# Вернуть к состоянию до последней миграции
$ sequelize-cli db:migrate:undo
# Вернуться к исходному состоянию
$ sequelize-cli db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js
# Сгенерировать seed
$ sequelize-cli seed:generate --name demo-user
# Запуск seed
$ sequelize-cli db:seed:all
# Откат последних seeders
$ sequelize-cli db:seed:undo
# Откат конкретного seed
$ sequelize-cli db:seed:undo --seed name-of-seed-as-in-data
# Отмена всех seeders
$ sequelize-cli db:seed:undo:all
```

---

# Objection

Objection.js — это ORM для Node.js, цель которого — не мешать вам и максимально упростить использование всей мощности SQL и базового механизма базы данных, сохраняя при этом общие вещи легко и приятно.

Objection.js построен на основе строителя SQL-запросов knex. Все базы данных, поддерживаемые knex, поддерживаются objection.js.

Что дает objection.js:

- простой декларативный способ определения моделей и отношений между ними;
- простой способ извлечения, вставки, обновления и удаления объектов с использованием всех возможностей SQL;
- механизмы быстрой загрузки, вставки и добавления графов объектов;
- простые в использовании транзакции;
- официальная поддержка TypeScript.

```bash
  $ yarn add objection knex pg
```

---

# Objection - модели

```ts {all|3|4-6|7-9|10-12|13-23|all}
import { Model } from 'objection';

class Person extends Model {
  static get tableName() {
    return 'persons';
  }
  static get idColumn() {
    return 'id';
  }
  fullName() {
    return this.firstName + ' ' + this.lastName;
  }
  static get jsonSchema() {
    return {
      type: 'object',
      required: ['firstName', 'lastName'],
      properties: {
        id: { type: 'integer' },
        parentId: { type: ['integer', 'null'] },
        //...
      }
    };
  }
}
```
--- 

```ts {all|3-5|7-18|all}
class Person extends Model {
  //...
  static get modelPaths() {
    return [__dirname];
  }

  static get relationMappings() {
    return {
      pets: {
        relation: Model.HasManyRelation,
        modelClass: 'Animal',  // or path.join(__dirname, 'Animal')
        join: {
          from: 'persons.id',
          to: 'animals.ownerId'
        }
      },
    };
  }
}
```

---

# Objection - отношения

Для того чтоб создать связь между моделям необходимо задать тип отношения: 
- Model.BelongsToOneRelation  - исходная модель имеет внешний ключ;
- Model.HasManyRelation       - связанная модель имеет внешний ключ;
- Model.HasOneRelation        - так же, как **HasManyRelation**, но для одной связанной строки;
- Model.ManyToManyRelation    - модель связана со списком других моделей через таблицу соединений;
- Model.HasOneThroughRelation - модель связана с одной моделью через таблицу соединений.

в статическом свойстве **relationMappings**

---

## OneToMany

```ts {all|4-13|all}
class Person extends Model {
  static tableName = 'persons';

  static relationMappings = {
    animals: {
      relation: Model.HasManyRelation,
      modelClass: 'Animal',
      join: {
        from: 'persons.id',
        to: 'animals.ownerId'
      }
    }
  };
}
```

---

## ManyToOne

```ts
class Person extends Model {
  static tableName = 'persons';

  static relationMappings = {
    animal: {
      relation: Model.HasOneRelation, // разница с прошлым примером
      modelClass: 'Animal',
      join: {
        from: 'persons.id',
        to: 'animals.ownerId'
      }
    }
  };
}
```

---

## ManyToMany

```ts
class Person extends Model {
  static tableName = 'persons';

  static relationMappings = {
    movies: {
      relation: Model.ManyToManyRelation,
      modelClass: 'Movie',
      join: {
        from: 'persons.id',
        through: {
          from: 'persons_movies.personId',
          to: 'persons_movies.movieId'
        },
        to: 'movies.id'
      }
    }
  };
}
```

---

# Objection - опции поиска

```ts
const middleAgedJennifers = await Person.query()
  .select('age', 'firstName', 'lastName')
  .where('age', '>', 40)
  .where('age', '<', 60)
  .where('firstName', 'Jennifer')
  .orderBy('lastName');
```

```sql
SELECT "age", "firstName", "lastName"
FROM "persons"
WHERE "age" > 40
AND "age" < 60
AND "firstName" = 'Jennifer'
ORDER BY "lastName" ASC
```

---

```ts
const people = await Person.query()
  .select('persons.*', 'parent.firstName as parentFirstName')
  .innerJoin('persons as parent', 'persons.parentId', 'parent.id')
  .where('persons.age', '<', Person.query().avg('persons.age'))
  .whereExists(
    Animal.query()
      .select(1)
      .whereColumn('persons.id', 'animals.ownerId')
  )
  .orderBy('persons.lastName');
```

```sql
SELECT "persons".*, "parent"."firstName" AS "parentFirstName"
FROM "persons"
INNER JOIN "persons"
  AS "parent"
  ON "persons"."parentId" = "parent"."id"
WHERE "persons"."age" < (
  SELECT avg("persons"."age")
  FROM "persons"
)
AND EXISTS (
  SELECT 1
  FROM "animals"
  WHERE "persons"."id" = "animals"."ownerId"
) ORDER BY "persons"."lastName" ASC
```

```ts

```

---

# Objection - транзакции

Транзакции так же деляться на управляемые и неуправляемые.

## Управляемая транзакция

```ts
// Глобальный knex
try {
  const returnValue1 = await Person.transaction(async trx => {
    //...
    return 'the return value of the transaction';
  });
} catch (err) {
  // Тут транзакция была отменена.
}
// Не глобальный
const returnValue2 = await Person.transaction(knex, async trx => { ... })
// Или можно использовать просто knex 
const returnValue3 = await knex.transaction(async trx => { ... })
```

Приведенный выше пример работает, если вы глобально установили экземпляр *knex* с помощью метода **Model.knex()**. Если вы этого не сделали, вы можете передать экземпляр knex в качестве первого аргумента метода транзакции или можно воспользоваться *knex*.

---

## Неуправляемая транзакция

```ts {all|1|5|7|all}
const trx = await Person.startTransaction();

try {
  //...
  await trx.commit();
} catch (err) {
  await trx.rollback();
  throw err;
}
```

---

# Objection - CLI

```bash {all|1-3|4-6|7-8|9-10|11-13|14-15|16-17|18-19|20-21|all}
  # Создание конфиг файла .knexfile
  $ knex init
  $ knex init -x ts
  # Создание миграции
  $ knex migrate:make migration_name 
  $ knex migrate:make migration_name -x ts
  # Откатить последний пакет миграций
  $ knex migrate:rollback
  # Откатить все миграции
  $ knex migrate:rollback --all
  # Запустить миграции
  $ knex migrate:up
  $ knex migrate:up 001_migration_name.js
  # Отменить последнюю выполненную миграцию
  $ knex migrate:down
  # Посмотреть завершенные и новые миграции
  $ knex migrate:list
  # Создание нового Seed
  $ knex seed:make seed_name
  # Запуск Seeders
  $ knex seed:run
```