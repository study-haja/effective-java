# Item2: 생성자에 매개변수가 많다면 빌더를 고려하라



생성자나 정적 팩터리 메소드에서는 매개변수가 많을 경우 개발자가 대응하기 어렵다는 제약이 있다. 이렇게 말로 표현하면 와닿지 않을 수 있다. 그래서 밑에 비디오 클래스를 예를 들어보려고 한다. 

비디오 클래스에는 다음과 같은 필드가 존재한다.

1. 비디오 아이디 (필수)
2. 비디오 타이틀 (필수)
3. 비디오 설명 (선택)
4. 비디오 등록 날짜 (선택)
5. 비디오 파일 (필수)
6. 비디오 썸네일 이미지 (선택)

단 필수로 받는 필드는 비디오 아이디, 타이틀, 비디오 파일이고, 선택으로 받는 필드는 비디오 등록 날짜, 썸네일 이미지, 설명이라고 정의 하자. 



## 점층적 생성자 패턴

프로그래머들은 점층적 생성자 패턴을 즐겨 사용할 것이다. 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수 2개를 받는 생성자등 이러한 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식이다. 이 부분을 밑에 코드로 표현해 보려고 한다.

```java
class Video {
  private final Long id;
  private final String title;
  private final String description;
  private final LocalDateTime createTime;
  private final File videoFile;
  private final File thumbnailFile;
  
  public Item2(Long id, String title, File videoFile) {
        this(id, title, "", videoFile);
    }

    public Item2(Long id, String title, LocalDateTime createTime, File videoFile) {
        this(id, title, "", createTime, videoFile);
    }

    public Item2(Long id, String title, String description, File videoFile) {
        this(id, title, description, LocalDateTime.now(), videoFile);
    }

    public Item2(Long id, String title, File videoFile, File thumbnailFile) {
        this(id, title, "", LocalDateTime.now(), videoFile, thumbnailFile);
    }

    public Item2(Long id, String title, String description, LocalDateTime createTime, File videoFile) {
        this(id, title, description, createTime, videoFile, null);
    }
  
  	... // 다른 매개변수 생성자

    public Item2(Long id, String title, String description, LocalDateTime createTime, File videoFile, File thumbnailFile) {
        this.id = id;
        this.title = title;
        this.description = description;
        this.createTime = createTime;
        this.videoFile = videoFile;
        this.thumbnailFile = thumbnailFile;
    }
}
```

이 예제를 보면 선택 매개변수 때문에 엄청난 생성자들이 생겨나는 것을 볼 수 있다. 필자도 헤깔려 생성자를 만들다가 그만 두었다. 지금은 선택 필드가 3개 뿐이지만 4개 5개가 되면? 그때는 걷잡을 수 없게 된다.



따라서 요약하면, 점층적 생성자 패턴도 쓸 수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다. 그 이유는 다음과 같다.

1. 코드를 읽을 때 각 값의 의미가 무엇인지 헷갈릴 것이고, 매개변수가 몇 개인지도 주의해서 세어 보아야 할 것 이다. 
2. 타입이 같은 매개변수가 연달아 늘어서 있으면 찾기 어려운 버그로 이어 질 수 있다. 
3. 클라이언트가 실수로 매개변수의 순서를 바꿔 건네줘도 컴파일러는 알아채지 못하고, 결국 런타임에 엉뚱한 동작을 하게 된다.



## 자바빈즈 패턴

이번에는 선택 매개변수가 많을 때 활용할 수 있는 두번째 대안 이다. 매개 변수가 없는 생성자로 객체를 만든 후, 세터(Setter) 메소드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.

```java
class Video {
  private Long id;
  private String title;
  private String description;
  private LocalDateTime createTime;
  private File videoFile;
  private File thumbnailFile;
  
  public Video() {}
  
   public void setId(Long id) {
        this.id = id;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public void setCreateTime(LocalDateTime createTime) {
        this.createTime = createTime;
    }

    public void setVideoFile(File videoFile) {
        this.videoFile = videoFile;
    }

    public void setThumbnailFile(File thumbnailFile) {
        this.thumbnailFile = thumbnailFile;
    }
}
```

이 패턴은 점층적 생성자 패턴의 단점들이 자바빈즈 패턴에서 더 이상 보이지 않는다. 코드가 길어지긴 했지만 인스턴스를 만들기 쉽고, 그 결과 더 읽기 쉬운 코드가 되었다.

```java
@Test
public void enrollVideo() {
  Video video = new Video();
  video.setId(1L);
  video.setTitle("title");
  video.setDescription("description");
  video.setCreatTime(LocalDateTime.now());
  ...
}
```

하지만 불행하게도 이 패턴은 객체 하나를 만들려면 메서드를 여러개 호출해야한다는 단점이 있다. 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다. 점층적 생성자 패턴은 매개변수들이 유효한지를 생성자에서 확인하면 일관성을 유지할 수 있었는데, 그 장치가 완전히 사라진 것이다. 이 일관성이 깨진 객체를 만들어지면, 버그를 심은 코드와 그 버그 때문에 런타임에 문제를 겪는 코드가 물리적으로 멀리 떨어져 있을 것이므로 디버깅도 만만치 않을 것이다. 

즉, 자바 빈즈 패턴은 클래스를 ***불변으로 만들 수 없으며*** 스레드 안정성을 얻으려면 프로그래머가 추가 작업을 해줘야 한다. 물론 이 단점을 완화하고자 수동으로 Freezing 방법을 사용하는데 이 방법은 복잡하고, 해당 메소드를 확실히 호출해줬는지 보증할 방법이 없어 실전에는 잘 사용하지 않는다.



## 빌더 패턴

위에 패턴들은 각각의 장단점이 상이했다. 점층적 생성자 패턴의 안정성과, 자바 빈즈 패턴의 가독성 이 두마리의 토끼를 잡을 수 있는 방법이 없을까??

이렇게 생각해 나온 패턴이 바로 빌더 패턴이다. 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 팩토리)를 호출해 빌더 객체를 얻는다. 다음 예를 보자.

```java
class Video {
  private final Long id;
  private final String title;
  private final String description;
  private final LocalDateTime createTime;
  private final File videoFile;
  private final File thumbnailFile;

  public static class Builder {
    //필수 매개변수
    private final Long id;
    private final String title;
    private final File videoFile;
        
    //선태 매개변수 - 기본 값으로 초기화
    private String description;
    private LocalDateTime createTime;
    private File thumbnailFile;

    public Builder(Long id, String title, File videoFile) {
      this.id = id;
      this.title = title;
      this.videoFile = videoFile;
    }
        
    public Builder description(String description) {
      this.description = description;
      return this;
    }

    public Builder createTime(LocalDateTime createTime) {
      this.createTime= createTime;
      return this;
    }

    public Builder thumbnailFile(File thumbnailFile) {
      this.thumbnailFile = thumbnailFile;
      return this;
    }
        
    public Item2 build() {
      return new Item2(this);
    }
  }
    
  private Item2 (Builder builder) {
    this.id = builder.id;
    this.title = builder.title;
    this.description = builder.description;
    this.createTime = builder.createTime;
    this.videoFile = builder.videoFile;
    this.thumbnailFile = builder.thumbnailFile;
  }
}
```

Video 클래스는 불변이며, 모든 매개변수의 기본값들을 한곳에 모아 뒀다. 빌더의 세터 메소드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출이 가능하다. 이런 방식을 이펙티브 자바에서는 플루언트 API 혹은 메소드 연쇄라고 말한다. 

다음은 비디오 클래스를 사용하는 클라이언트 코드의 모습이다.

```java
@Test
public void enrollTest() {
  Item2 item2 = new Item2.Builder(1L, "title", new File("./videoFile.mp4"))
    .createTime(LocalDateTime.now())
    .description("description")
    .thumbnailFile(null)
    .build();
  ...
}
```

이 클라이언트 코드를 보면 쓰기 쉽고, 무엇보다 읽기 쉽다! 즉 빌더패턴을 통해 두마리의 토끼를 잡은 셈이다.

 핵심이 도드라져 보이도록 유효성 검사 코드는 생략했다. 잘못된 매개변수를 최대한 일찍 발견하려면 빌더의 생성자와 메소드에서 입력 매개변수를 검사하고  build 메소드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식을 검사하면 된다. 그리고 공격에 대비해 이런 불변식을 보장하려면 빌더로 부터 매개변수를 복사한 후 해당 객체 필드들도 검사해야 한다. 만약 검사해서 잘못된 점을 발견하면 어떤 매개변수가 잘못 되었는지를 자세히 알려주는 메세지를 담아 IllegalArgumentException을 던지면 된다.

빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.  각 계층의 클래스에 관련 빌더를 멤버로 정의하자. 추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다. 다음 예를 보자.

```java
public abstract class Video {
  public enum Tagging { ACTION, COMEDY, ENTERTAINMENT, SF, DOCUMENT, CLASSIC }
  final Set<Tagging> taggings;
  final String title;
  
  abstract static class Builder<T extends Builder<T>> {
    EnumSet<Tagging> taggings = EnumSet.noneOf(Tagging.class);
    String title;
    
    public T addTagging(Tagging tagging) {
      taggings.add(Objects.requireNonNull(tagging));
      return self();
    }
    
    public T addTitle(String title) {
      this.title = title;
      return self();
    }
    
    abstract Video build();
    
    protected abstract T self();
  }
  
  Video(Builder<?> builder) {
    taggings = builder.taggings.clone();
    title = builder.title;
  }
 
}
```

Video.Builder 클래스는 재귀적 타입 한정을 이용하는 제네릭 타입이다. 여기서 추상 메소드인 self를 더해 하위 클래스에서는 형변환하지 않고도 메소드 연쇄를 지원할 수 있다. 

다음 Video 하위 클래스 2개를 만들어보자. 하나는 유투브 비디오 이고 다른 하나는 넷플릭스 비디오 인다. 유투브 비디오에는 필수 매개변수가 채널, 비디오 URL 이고 넷플릭스의 필수 매개변수는 비디오 URL, 시리즈라고 하자.

* YoutubeVideo.class

```java
public class YoutubeVideo extends Video {
  public enum Channel { CHANNAL_A, CHANNEL_B, CHANNEL_C }
  private final Channel channel;
  private final String videoUrl;

  public static class Builder extends Video.Builder<Builder> {
    private final Channel channel;
    private final String videoUrl;

    public Builder(Channel channel, String videoUrl) {
      this.channel = Objects.requireNonNull(channel);
      this.videoUrl = videoUrl;
    }

    // YoutubeVideo 클래스 반환
    @Override
    public YoutubeVideo build() {
      return new YoutubeVideo(this);
    }

    @Override
    protected Builder self() {
      return this;
    }
  }

  private YoutubeVideo(Builder builder) {
    super(builder);
    channel = builder.channel;
    videoUrl = builder.videoUrl;
  }
}
```

* NetFlixVideo.class

```java
public class NetFlixVideo extends Video {
	private final List<Video> series;
  private final String videoUrl;
  
  public static class Builder extends Video.Builder<Builder> {
    private final List<Video> series = new ArrayList<>();
    private final String videoUrl;
    
    public Builder(String videoUrl) {
      this.videoUrl = videoUrl;
    }
    
    public Builder addSeries(Video video) {
      this.series.add(video);
      return this;
    }
    
    // NetFlixVideo 클래스 반환
    @Override
    public NetFlixVideo build() {
      return new NetFilxVideo(this);
    }
    
    @Override
    protected Builder self() {
      return this;
    }
  }
  
  private NetFlixVideo(Builder builder) {
    super(builder);
    series = builder.series;
    videoUrl = builder.videoUrl;
  }
}  
```

 각 하위 클래스의 빌더가 정의한 build 메소드는, 해당하는 구체 하위 클래스를 반환하도록 선언했다. YoutubeVideo.Builder는 YoutubeVideo를 반환 하고, NetFlixVideo.Builder는 NetFlix를 반환하도록 한 것이다. 이렇게 반환하는 기능을 공변 반환 타이핑이라 한다. 이 기능을 이용하면 클라이언트가 형변환에 신경 쓰지 않고고 빌더를 사용할 수 있다.

 이러한 '계층적 빌더'를 사용하는 클라이언트의 코드도 앞선 예제와 별 다를게 없다. 다음 예를 보자. (물론 열거 타입은 상수로 정적 임포트 했다고 가정한다.)

```java
@Test
public void enrollVideo() {
  YoutubeVideo youtubeVideo = new YoutubeVideo.Builder(CHANNAL_A, "https://youtube.api....")
    .addTagging(ENTERTAINMENT).addTagging(COMEDY).addTitle("youtube title").build();
  
  NetFlixVideo netflixVideo = new NetFlixVideo.Builder("https://netflix.api....")
    .addSeries(BEFORE_VIDEO).addTagging(ACTION).addTitle("netflix title").build();
}
```

클라이언트 코드도 앞선 비디오 예제와 같이 빌더를 사용하는 코드와 다르지 않다. 똑같이 필수로 받는 매개변수를 받고, 비디오의 공통 기능인 태깅과 타이틀을 상위클래스로 빼서 확장에 유연하게 대처했다. 

 빌더 패턴은 상당히 유연하다. 빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수 있다. 객체에 부여되는 일련번호와 같은 특정 필드는 빌더가 알어서 채우도록 할 수 있다.

 하지만 빌더 패턴도 다음과 같이 단점이 존재한다. 

1. 객체를 만들려면 그에 앞서 빌더부터 만들어야 한다. - 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.
2. 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다.

첫번째는 하드웨어 성능상 판단을 하면 될것 같고 두번째는 사실 API가 시간이 지날수록 매개변수가 많아지는 경향이 있으니 그때에 맞게 사용하면 된다.



 물론 생성자나 정적 팩터리 방식으로 시작했다가 나중에 매개변수가 많아지면 빌더 패턴으로 전환 할 수 있다. 하지만 수정 시간 비용이 많이 발생할 것이다. 그러니 애초에 빌더로 시작하는 편이 나을 때가 많다. 결론을 말하자면 생성자나 정적 팩토리가 처리해야할 매개변수가 많다면, **가독성과, 안전성**을 겸비한 빌더 패턴을 사용하는게 훨씬 좋다.