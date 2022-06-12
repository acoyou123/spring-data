# Spring Data

## Spring Data とは

- データストア(※1)に対するアクセスを抽象化するライブラリで、異なるデータストアに対するアクセスを統一化する
- 各データストアをRepository(※2)というインタフェースに抽象化し、オブジェクトを操作することで、データストアの操作を行う
- インタフェース（[spring-data-commons](https://github.com/spring-projects/spring-data-commons)が提供）や、RDBやMongo等に対応する各種実装（spring-data-jpaやspring-data-mongodb）が提供されている

※1：RDBだけでなく、NoSQLもサポートされている<br>
※2：Repositoryとは、エンティティオブジェクトに対して、言わばコレクションのように振る舞うインタフェース

## 主なインタフェース

### [CrudRepository](https://spring.pleiades.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)

CRUD（Create Read Update Delete） 機能を提供する

```Java
public interface CrudRepository<T, ID> extends Repository<T, ID> {

  <S extends T> S save(S entity); ...(1)

  Optional<T> findById(ID primaryKey); ...(2)

  Iterable<T> findAll(); ...(3)

  long count(); ...(4)

  void delete(T entity); ...(5)

  boolean existsById(ID primaryKey); ...(6)

  // … more functionality omitted.
}
```

1. 指定されたエンティティを保存する
2. 指定された ID で識別されるエンティティを返す
3. すべてのエンティティを返す
4. エンティティの数を返す
5. 指定されたエンティティを削除する
6. 指定された ID のエンティティが存在するかどうかを示す

### [PagingAndSortingRepository](https://spring.pleiades.io/spring-data/commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)

CrudRepositoryの拡張により、ページネーションとソートの抽象化を使用してエンティティを取得する追加のメソッドが提供

```Java
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

  Iterable<T> findAll(Sort sort);

  Page<T> findAll(Pageable pageable);
}
```

20 のページサイズで User の 2 番目のページにアクセスするには、次のようなことができる。

```Java
PagingAndSortingRepository<User, Long> repository = // … get access to a bean
Page<User> users = repository.findAll(PageRequest.of(1, 20));
```

## クエリメソッド

通常、標準の CRUD 機能リポジトリには、基になるデータストアに対するクエリがある
Spring Data では、これらのクエリを宣言することは 4 ステップのプロセスが必要

1. リポジトリを宣言し、処理するドメインクラスと ID タイプに入力

```Java
interface PersonRepository extends Repository<Person, Long> { … }
```

2. インターフェースでクエリメソッドを宣言

```Java
interface PersonRepository extends Repository<Person, Long> {
  List<Person> findByLastname(String lastname);
}

```

3. JavaConfig または XML 構成を使用して、Spring をセットアップし、これらのインターフェースのプロキシインスタンスを作成

4. リポジトリインスタンスを挿入して使用

```Java
class SomeClient {

  private final PersonRepository repository;

  SomeClient(PersonRepository repository) {
    this.repository = repository;
  }

  void doSomething() {
    List<Person> persons = repository.findByLastname("Matthews");
  }
}
```

## 主な関連ライブラリ

### [Spring Data JPA](https://spring.pleiades.io/spring-data/jpa/docs/current/reference/html/#reference)

- JPA（Java Persistence API ※3）のSpring Dataリポジトリサポート
- JPA ベースのリポジトリを簡単に実装できる
- 便利で高機能（複雑になりがち）
    - 遅延ロード
    - エンティティのプロキシ
    - セッション / 1st レベルキャッシュ
    - エンティティの監視
- カスタムクエリの記法はJPQL(Java Persistence Query Language)

※3：JPAは、Java標準のORM（Object-Relational Mapping：オブジェクトとデータベースのマッピング）

### [Spring Data JDBC](https://spring.pleiades.io/spring-data/jdbc/docs/current/reference/html/#reference)

- JDBC（Java Database Connectivity ※4）のSpring Dataリポジトリサポート
- Spring Data JPAよりもシンプルで分かりやすいモジュールとして後から公開された
    - Spring Data JPAのような文法でRepositoryを定義すると、[NamedParameterJdbcTemplate](https://spring.pleiades.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-NamedParameterJdbcTemplate)を利用してSQLが実行される
- キャッシング、遅延読み込み、後書き、または JPA のその他の多くの機能はされていない

※4：JDBCは、JavaでDBに接続するためのAPI

## Spring BootでのDB接続

|  | 概要 |
|---|---|
| JDBC | JavaでDBに接続するための標準API<br>※DB操作には接続・クローズの処理を書く必要がある<br>※SQLは自前で書く必要がある |
| Spring JDBC |	JDBCの使いづらい部分を補うフレームワーク<br>application.propertiesにDB接続情報定義するだけで接続可能<br>※SQLは自前で書く必要がある |
| JPA |	Java標準のORMの仕様<br>JDBC系と異なりSQLを自動生成し、ORMでデータ取得結果をEntityクラスに変換してくれる |
| Spring Data JPA	| JPAを使いやすくする抽象クラス/インタフェースを提供 |
| Spring Data JDBC	| Spring Data JPAよりもシンプルで分かりやすいモジュールとして後から公開された |
