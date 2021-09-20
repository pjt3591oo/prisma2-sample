# prisma2

### 패키지 설치 & 프로젝트 셋업

* 프로젝트 생성

```bash
$ npm init -y
```

* 패키지 설치

```bash
$ npm i -D typescript ts-node @types/node
```

```bash
$ npm install @prisma/client
```

* tsconfig.json 생성

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "outDir": "dist",
    "strict": true,
    "lib": ["esnext"],
    "esModuleInterop": true
  }
}
```

### init

```bash
$ npx prisma init
```

.gitignore, .env, prisma/schema.prisma 생성

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

data source, generator, data model 정의한다.

```prisma
datasource db {
  provider = "mysql"
  url      = "mysql://root:password@localhost:3306/prisma_test"
}
```

### studio

```bash
$ npx prisma studio
```

### db

* 데이터베이스 정보로 스키마 생성

```bash
$ npx prisma db pull
```

* 마이그레이션 생성없이 schema.prisma 정보로 데이터베이스 업데이트

```bash
$ npx prisma db push
```

### migrate

* 마이그레이션 생성

마이그레이션 생성 시 이전에 생성한 마이그레이션이 적용되지 않았으면 이전 마이그레이션을 적용한 후 새로운 마이그레이션을 생성한다

```bash
$ npx prisma migrate dev --name [마이그레이션 이름] --create-only
```

* 마이그레이션 적용

로컬환경에서 사용

dev의 경우 schema를 보고 변경사항이 있으면 마이그레이션 생성을 시도함 => 마이그레이션 적용 시 스키마 변경사항이 있으면 마이그레이션 생성 후 생성된 마이그레이션도 적용됨

```bash
$ npx prisma migrate dev
```

운영서버 사용 - production staging server, test staging server 등

deploy의 경우 schema를 보지 않고 생성된 마이그레이션만 적용한다 => 스키마 변경사항이 있더라도 생성된 마이그레이션에 대해서만 적용
(production, test stagingd의 경우 소스코드를 직접적으로 수정하지 않기 때문에 deploy 사용권장)

추가로 deploy 수행시 10초동안 다른 명령어 수행을 잠금다고 한다.

```bash
$ npx prisma migrate deploy 
```

* reset

데이터 초기화 후 seed 데이터 생성

```bash
$ npx prisma migrate reset
```

* resolve

```bash
$ npx prisma migrate resolve --applied [마이그레이션]

$ npx prisma migrate resolve --rolled-back [마이그레이션]
```

### generate

스키마가 변경될 때마다 다음 명령어 수행을 해야함

./node_modles/prisma/client에 prisma.schema.prisma를 js로 변환해서 저장

변환된 js를 import하여 ORM을 사용하면 됨

```bash
$ npx prisma generate

Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma

✔ Generated Prisma Client (3.0.2) to ./node_modules/@prisma/client in 77ms
You can now start using Prisma Client in your code. Reference: https://pris.ly/d/client
```
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()
```

* watch 모드

```bash
$ npx prisma generate --watch
```

### @prisma/client

generate 수행 후 ./node_modules/@prisma/client에 js로 변화된 파일 생성

schema.prisma에 정의된 model은 js로 변환되어 PrismaClient 클래스를 통해 제공한다.

```js
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@prisma.io',
      posts: {
        create: { title: 'Hello World' },
      },
      profile: {
        create: { bio: 'I like turtles' },
      },
    },
  })
  const allUsers = await prisma.user.findMany({
    include: {
      posts: true,
      profile: true,
    },
  })
  console.dir(allUsers, { depth: null })
}

main()
  .catch((e) => {
    throw e
  })
  .finally(async () => {
    await prisma.disconnect()
  });
```

