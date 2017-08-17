## 로컬과 s3에 파일 업로드하기

### 필요 gem
```ruby
gem 'fog-aws' # aws s3 업로드 가능
gem 'mini_magick' # 이미지 크기 조정 # imagemacik이 필요함
gem 'carrierwave' # 파일업로드
```

`mini_magick`을 설치하기 위해서는 다음과 같은 작업이 필요하다
```bash
$ sudo apt-get update
$ sudo apt-get install imagemagick
```

### file 입력 받기
`multipart`가 true 이여야한다
```html
<%= form_for(@post, html: {multipart: true}) do |f| %>
```

### image uploader 만들기
```bash
$ rails g uploader image
```
몇 가지 주석을 해제한다
```ruby
class ImageUploader < CarrierWave::Uploader::Base

  # Include RMagick or MiniMagick support:
  # include CarrierWave::RMagick
  include CarrierWave::MiniMagick # 이미지 사이즈 조절

  # Choose what kind of storage to use for this uploader:
  storage :file
  # storage :fog

  # Override the directory where uploaded files will be stored.
  # This is a sensible default for uploaders that are meant to be mounted:
  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  # Create different versions of your uploaded files:
  # 이미지를 업로드할 때 원본말고도 thumb 버젼으로 업로드를 하나 더 한다는 것
  version :thumb do
    process resize_to_fit: [50, 50] # 비율 그대로 가져옴
    # process resize_to_fill: [50, 50] # 입력된 수치로 절대적으로 파일 크기를 수정함
  end
end
```
### model에 사진을 넣기 위한 준비
다음을 `/config/initializers/carrierwave.rb` 파일을 추가한 후 붙여넣는다
```ruby
require 'carrierwave/orm/activerecord'
```
### model 생성
```bash
$ rails g model title:string content:string image:string
```
이미지 업로드 할 model에 다음을 추가한다  
이때 model의 `image`와 밑의 `image`가 동일한 이름이여야 한다
```ruby
class User < ActiveRecord::Base
  mount_uploader :image, ImageUploader
end
```

### 사용하기
정상적으로 업로드가 되었다면 다음과 같은 방법으로 사진을 출력 할 수 있다
```ruby
<%= image_tag post.image.url %>
```
version이 추가되어있다면 다음과 같은 방법으로 사진을 출력 할 수 있다
`thumb`는 uploader에 version으로 지정한 이름을 써주면 된다
```ruby
<%= image_tag post.thumb.image.url %>
```
예를들어 likelion 버전의 url을 가져오고 싶으면  
`post.likelion.image.url`이 된다


### aws로 업로드하기
`/config/initializers/carrierwave.rb` 파일에 다음의 내용을 추가한다
```ruby
CarrierWave.configure do |config|
  config.fog_provider = 'fog/aws'            
  config.fog_credentials = {
    provider:              'AWS',                     
    aws_access_key_id:     "#{ENV['AWS_KEY']}",                   
    aws_secret_access_key: "#{ENV['AWS_SECRET']}",               
    region:                'ap-northeast-2', 
  }
  config.fog_directory  = 'name_of_directory'                   
end
```
환경변수는 다음과 같이 등록이 가능하다
```bash
$ echo "export AWS_KEY=키 값" >> ~/.profile
$ echo "export AWS_SECRET=키 값" >> ~/.profile
$ source ~/.profile
```
다음과 같이 입력했을때 값이 뜬다면 성공
```bash
$ echo $AWS_KEY
```
image uploader의 `storage`를 `fog`로 변경한다
```ruby
# Choose what kind of storage to use for this uploader:
# storage :file
storage :fog
```