# Talkbank
 
 ## Длина промокодов 
  
Длина Промокодов может быть как разной, так одинаковой. 
- Для цифрового промокода можно сделать длину прокода 8 цифр. Так как цифр всего 10, то и вариантов будет меньше, чем у буквеноцифрового, поэтому использовать не менее 8, на выходе получаем  100 млн. вариантов кодов. что обеспечит запас на долго.
- для буквеноцифрового промокода можно сделать прокод в 6 знаков, так как порядок вариантов для уникальных значений будет выше чем у цифрового.
 
 ## Классы и интерфейсы
 
 Описать интерфейс купонов, создать от него базовый класс, а конечные классы промокодов наследовать от базового. Реализовать в них логику по их генерации (Насыщенная модель), так мы сможем добавлять новые типы промокодов, просто добавив новый класс промокода, создаем новый тип и всю логику его генерации оставляем внутри.
 
 Для создания купонов лучше воспользоваться фабричным методом и в зависимости от типа, создавать объект промокода (class_exist можно использовать). 
 
 Недоступными свойствами лучше сделать текущее число попыток, чтобы клиент не мог подменить данные. 
 
 ```
 interface PromocodeInterface
{
    public function getDiscount(): float;

    public function getCode(): string;

    public function getMaxAttempt(): int;
    
    public function generate(): string;
}

abstract class BasePromocode implements PromocodeInterface
{
    const TYPE = 'string';

    /** @var string */
    public $code;

    /** @var float */
    public $discount;

    /** @var int */
    public $maxAttempt;

    /** @var int */
    protected $attempt;

    public function getType(): string
    {
        return static::TYPE;
    }
    
    //добавить геттеры и сеттеры

    abstract public function generate(): string;

}

class PromoString extends BasePromocode
{
    public function generate(): string
    {
        //возвращаем строку из букв и цифр, 6 знаков
    }
}

class PromoNumber extends BasePromocode
{
    const TYPE = 'number';
    
    public function generate(): string
    {
        //возвращаем строку из цифр, 8 знаков
    }
}

class PromocodeDto
{
    /** @var string */
    public $type;

    /** @var float */
    public $discount;

    /** @var int */
    public $maxUse;
}

class PromocodeFactory
{
    /**
     * @param string $type
     *
     * @throw PromocodeNotFoundTypeException
     *
     * @return PromocodeInterface
     */
    public function build(string $type): PromocodeInterface
    {

    }
}

class PromocodeService
{
    /**
     * @param PromocodeDto $promoDto
     *
     * @return string
     */
    public function generate(PromocodeDto $promoDto): string
    {

    }

    /**
     * @param string $code
     *
     * @throw PromocodeNotValidException
     *
     * @return float
     */
    public function apply(string $code): float
    {

    }

}
 
 ```
 
 ## Табличка в БД
  
Автоинкрементный ключ можно не делать, можно сам код сделать первичным ключом, так как нам не надо повторяющихся кодов, и также индекс по нему нужен, для более быстрой выборки по нему
  
 ```
 create table procodes
(
	code int not null,
	discount float not null,
	max_attempt int not null,
	attempt int default 0 null,
	type varchar(100) not null
);

create unique index procodes_code_uindex
	on procodes (code);

alter table procodes
	add constraint procodes_pk
		primary key (code);
```
 
 ## Апи
  
  По всем лучше использовать json, так как легкий, понятный и легкочитаемый формат.
  
 Создание купона
 
 ```
 POST /api/sale/coupon/create/
 Content-Type: application/json
 
{
  "type": "string",
  "discount": 10.5,
  "maxAttempt": 2
}
```
Response

```
200 OK

{
  "code": "string"
}

400 BAD REQUEST

{
  "message": "error text"
}
```

  
 Применение купона. Думаю напрямую использовать GET запрос опасно, так как потому что он не просто возвращает скидку, но и увеличивает кол-во его применений.
 
 ```
 PATCH /api/sale/coupon/apply/{code}
 Content-Type: application/json
 ```
 
 Response

```
200 OK

{
  "discount": "float"
}

404 NOT FOUND

{
  "message": "error text"
}
```
