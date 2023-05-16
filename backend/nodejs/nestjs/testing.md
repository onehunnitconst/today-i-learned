# 프레임워크

## Jest

[메타 (페이스북)](https://github.com/facebook/jest)에서 만드는 자바스크립트 테스트 프레임워크입니다. NestJS에서 기본 테스트 프레임워크로 사용합니다.

## ts-mockito

자바의 모킹 프레임워크인 Mockito에서 영감을 받아 제작된 타입스크립트용 모킹 프레임워크입니다. Jest는 기본적으로 자바스크립트 프레임워크이고 타입스크립트를 지원하지만 ts-mockito는 타입스크립트에서 사용할 것을 상정하고 설계된 프레임워크입니다.

물론 NestJS의 공식 문서에서는 jest의 모킹 기능을 이용한 예제를 제시하고 있지만, 저희는 Jest의 모킹 기능이 아니라 타입스크립트 환경에 더욱 최적화된 ts-mockito를 사용하도록 하겠습니다.

## faker.js

가짜 데이터를 생성해주는 자바스크립트 프레임워크입니다. 

# 유닛 테스트

코드가 잘 작동하는 지 확인하는 가장 좋은 방법은 어플리케이션을 구동하여 테스트해보는 방법일 것입니다. 하지만, 어플리케이션의 규모가 커질수록 새로운 기능을 일일이 어플리케이션을 켜고 테스트할 수는 없는 노릇입니다. 마치 핸들이 잘 만들어졌는지 테스트하기 위해 일일이 자동차에 핸들을 장착하고 자동차에 시동을 건 다음 시운전을 하는 것과 마찬가지입니다.

새로 개발한 로직만을 독립적인 환경에서 테스트할 수 있도록 하는 것이 바로 **유닛 테스트**입니다. NestJS에서는 유닛 테스트를 위한 도구를 자체적으로 제공하고 있습니다.

예시를 보며 유닛 테스트 방법을 익히겠습니다. 아래는 LUMOS 프로젝트의 `items.service.spec.ts`에서 코드 예시를 따왔습니다.

## TestingModule 정의

```tsx
import { Test, TestingModule } from '@nestjs/testing';
import { DataSource, Repository } from 'typeorm';
import { Item } from './entities/item.entity';
import { ItemStock } from './entities/item-stock.entity';
import { mock, instance } from 'ts-mockito';

describe('Items Test suite', () => {
  let service: ItemsService;

  const itemsRepository = mock<Repository<Item>>();
  const itemStockRepository = mock<Repository<ItemStock>>();
  const dataSource = mock<DataSource>();

  // 모든 테스트 전 실행
  beforeEach(async () => {
    // mock 인스턴스를 UsersService에 대신 주입
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        {
          provide: ItemsService,
          useFactory: () => {
            return new ItemsService(
              instance(dataSource),
              instance(itemsRepository),
              instance(itemStockRepository),
            );
          },
        },
      ],
    }).compile();

    service = module.get<ItemsService>(ItemsService);
  });

  describe('모듈 정의', () => {
    it('should be defined', () => {
      expect(service).toBeDefined();
    });
  });
});
```

NestJS에서 유닛 테스트를 하기 위해서는 먼저 어플리케이션과 독립된 TestingModule 정의가 필요합니다. 아시다시피 서비스에는 데이터베이스와 연동하기 위한 **TypeORM 리포지토리**와 **데이터 소스**를 의존성으로 주입하고 있습니다. 유닛 테스트에서는 해당 인프라스트럭처 코드를 대체하여 테스트를 해야하므로, 이를 대체할 모킹 클래스와 그 인스턴스를 주입합니다.

### 클래스 모킹

```tsx
const itemsRepository = mock<Repository<Item>>();
const itemStockRepository = mock<Repository<ItemStock>>();
const dataSource = mock<DataSource>();
```

ts-mockito의 mock 함수를 이용하여 먼저 클래스를 모킹합니다.

### 인스턴스 주입

```tsx
beforeEach(async () => {
  // mock 인스턴스를 UsersService에 대신 주입
  const module: TestingModule = await Test.createTestingModule({
    providers: [
      {
        provide: ItemsService,
        useFactory: () => {
          return new ItemsService(
            instance(dataSource),
            instance(itemsRepository),
            instance(itemStockRepository),
          );
        },
      },     
    ],
  }).compile();

  service = module.get<ItemsService>(ItemsService);
});
```

위에서 선언한 모킹 클래스의 인스턴스를 ItemService에 주입합니다. module.get() 함수를 이용하여 해당 모킹 모듈들이 주입된 서비스의 인스턴스를 꺼내올 것입니다.

### 모듈 정의 확인

```tsx
describe('모듈 정의', () => {
  it('should be defined', () => {
    expect(service).toBeDefined();
  });
});
```

모듈이 잘 정의되었는지 확인합니다. `describe`는 여러 개의 테스트 단위를 묶어주는 역할이며, `it`은 하나의 테스트 단위로 이해하시면 됩니다. 

`it` 에서 모듈이 정의되었는지 확인하기위해 `expect().toBeDefined()`를 호출합니다.

### 테스트 실행

```bash
yarn test src/app-general/items/items.service.spec.ts
```

## 기능 테스트

위에서 TypeORM 리포지토리와 데이터 소스를 모킹 클래스로 대체한다고 하였습니다. 따라서 모킹 클래스로 생성된 인스턴스는 빈 껍데기입니다.

비즈니스 로직에서 호출된 코드를 대신하기 위하여 ts-mockito의 **when** 함수를 이용하여 클래스의 메소드들을 흉내낼 것입니다.

아래는 전체 테스트 코드입니다.

```tsx
import { Test, TestingModule } from '@nestjs/testing';
import { DataSource, Repository } from 'typeorm';
import { Item } from './entities/item.entity';
import { ItemsService } from './items.service';
import { anything, instance, mock, when } from 'ts-mockito';
import { ItemStock } from './entities/item-stock.entity';
import { CreateItemDto } from './dto/create-item.dto';
import { faker } from '@faker-js/faker';
import { randomUUID } from 'crypto';
import { ItemDto } from './dto/item.dto';

describe('Items Test suite', () => {
  let service: ItemsService;
  let data: Item[];

  // ts-mockito를 통한 mock 생성
  const itemsRepository = mock<Repository<Item>>();
  const itemStockRepository = mock<Repository<ItemStock>>();
  const dataSource = mock<DataSource>();

  const generateCreateItemDto = (): CreateItemDto => ({
    itemCode: randomUUID(),
    name: faker.commerce.product(),
  });

  // 모든 테스트 전 실행
  beforeEach(async () => {
    // mock 인스턴스를 UsersService에 대신 주입
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        {
          provide: ItemsService,
          useFactory: () => {
            return new ItemsService(
              instance(dataSource),
              instance(itemsRepository),
              instance(itemStockRepository),
            );
          },
        },
      ],
    }).compile();

    service = module.get<ItemsService>(ItemsService);
    data = [];

    // 기존 모듈 메소드를 모킹
    when(itemsRepository.create(anything())).thenCall((form: CreateItemDto) => {
      return {
        ...form,
        createdAt: new Date(),
        updatedAt: new Date(),
        stock: [],
      };
    });

    when(itemsRepository.save(anything())).thenCall(async (item: Item) => {
      const savedItem = { ...item }
      data.push(savedItem);
      return savedItem;
    });
  });

  describe('모듈 정의', () => {
    it('should be defined', () => {
      expect(service).toBeDefined();
    });
  });

  describe('제품 생성 및 조회', () => {
    it('제품 생성', async () => {
      const form = generateCreateItemDto();
      const item = await service.createItem(form);

      expect(item.itemCode).toBe(form.itemCode);
      expect(item.name).toBe(form.name);
    });

    it('제품 단건 조회', async () => {
      const form = generateCreateItemDto();
      const item = await service.createItem(form);

      when(itemsRepository.findOne(anything())).thenResolve(item);
      const findItem = await service.findOne(item.itemCode);

      expect(findItem).toEqual(ItemDto.fromEntity(item));
    });

    it('제품 목록 조회', async () => {
      const itemLength = 5;

      await Promise.all([ ...Array(itemLength).keys() ].map(async (_) => {
        const form = generateCreateItemDto();
        return await service.createItem(form);
      }));

      when(itemsRepository.find()).thenResolve(data);
      const findItems = await service.findItems();

      expect(findItems.length).toBe(itemLength);
    });
  });
});t
```

### 제품 생성 테스트 예시

제품을 생성하는 서비스 로직은 다음과 같이 되어있습니다.

```tsx
async createItem(form: CreateItemDto) {
  const newItem = this.itemsRepository.create({
    ...form,
  });

  return await this.itemsRepository.save(newItem);
}
```

여기서 우리는 `create`와 `save` 메서드를 흉내내어야 합니다. `when` 메서드를 이용하여 다음과 같이 작성합니다.

```tsx
when(itemsRepository.create(anything())).thenCall((form: CreateItemDto) => {
  return {
    ...form,
    createdAt: new Date(),
    updatedAt: new Date(),
    stock: [],
  };
});

when(itemsRepository.save(anything())).thenCall(async (item: Item) => {
  const savedItem = { ...item }
  data.push(savedItem);
  return savedItem;
})
```

해당 예시에서는 모든 테스트에서 공통적으로 쓰이는 모듈이므로 `beforeEach` 내부에 정의하였습니다.

이렇게 메서드를 흉내낸 다음 테스트 코드를 다음과 같이 작성합니다.

```tsx
describe('제품 생성 및 조회', () => {
  it('제품 생성', async () => {
		// given
    const form = generateCreateItemDto();

		// when
    const item = await service.createItem(form);

		// then
    expect(item.itemCode).toBe(form.itemCode);
    expect(item.name).toBe(form.name);
  });
  ...
});
```

텍스트를 작성할 때는 **Given-When-Then (GWT)** 양식을 따라서 작성하는 것이 좋습니다. GWT의 구분은 다음과 같습니다.

| 구분 | 설명 | 예시에서 |
| --- | --- | --- |
| Given | 테스트 전 주어진 초기값 혹은 초기 상태.  | 예시에서는 createItem의 파라미터로 들어갈 CreateItemDto 객체입니다. |
| When | 테스트 수행 중 실행되는 기능.  | 예시에서는 서비스의 createItem 메서드입니다. |
| Then | 테스트 결과.  | 예시에서는 expect().toBe()를 이용하여 초기값인 form과 When 단계에서 처리된 item의 값을 비교합니다 |