# Lời giới thiệu

> Các mô hình về vai trò là rất quan trọng. <br/>
> -- Alex J. Murphy / thành viên RoboCop

Mục đích của tài liệu này là trình bày các `best practices và style` cho
việc phát triển ứng dụng với Rails 4. Đây là bản bổ sung cho
[Ruby coding style guide](https://github.com/bbatsov/ruby-style-guide).

Một vài mục chỉ áp dụng được với Rails 4.0+.

Bạn có thể xuất ra `PDF` hay `HTML` bằng công cụ
[Pandoc](http://pandoc.org/).

Các bản dịch cho ngôn các ngữ khác:

* [Tiếng Trung Giản thể](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
* [Tiếng Trung Phồn thể](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhTW.md)
* [Tiếng Đức](https://github.com/arbox/de-rails-style-guide/blob/master/README-deDE.md)
* [Tiếng Nhật](https://github.com/satour/rails-style-guide/blob/master/README-jaJA.md)
* [Tiếng Nga](https://github.com/arbox/rails-style-guide/blob/master/README-ruRU.md)
* [Tiếng Thổ Nhĩ Kỳ](https://github.com/tolgaavci/rails-style-guide/blob/master/README-trTR.md)
* [Tiếng Hàn](https://github.com/pureugong/rails-style-guide/blob/master/README-koKR.md)
* [Tiếng Việt](https://github.com/CQBinh/rails-style-guide/blob/master/README-viVN.md)

# Hướng dẫn cách viết code Rails (Rails style)

Bản hướng dẫn này khuyến nghị những phương án tốt nhất, do đó mọi người có thể
đọc, hiểu, bảo trì code của nhau dễ dàng hơn. Những style này đều lấy từ nhu cầu
thực tế, và nó sẽ giúp cuộc sống lập trình viên dễ thở hơn.

Những luật đã cũ rồi thì tôi không đề cập đến trong này, và hầu hết các luật
đều dựa trên những hiểu biết của tôi trong lĩnh vực lập trình phần mềm
chuyên nghiệp. Những phản hồi, góp ý, đóng góp luôn được hoan nghênh.

## Mục lục

* [Configuration](#configuration)
* [Routing](#routing)
* [Controllers](#controllers)
  * [Rendering](#rendering)
* [Models](#models)
  * [ActiveRecord](#activerecord)
  * [ActiveRecord Queries](#activerecord-queries)
* [Migrations](#migrations)
* [Views](#views)
* [Internationalization](#internationalization)
* [Assets](#assets)
* [Mailers](#mailers)
* [Active Support Core Extensions](#active-support-core-extensions)
* [Time](#time)
* [Bundler](#bundler)
* [Managing processes](#managing-processes)

## Configuration

* <a name="config-initializers"></a>
  Những `custom initialization` thì đặt ở thư mục `config/initializers`.
  Những code này sẽ được load một lần duy nhất khi khởi động rails app.
<sup>[[link](#config-initializers)]</sup>

* <a name="gem-initializers"></a>
  Mỗi `initialization code` của từng `gem` khác nhau thì nên đặt ở từng file
  khác nhau, và đặt tên tương ứng với tên gem, vd: `carrierwave.rb`, `active_admin.rb`...
<sup>[[link](#gem-initializers)]</sup>

* <a name="dev-test-prod-configs"></a>
  Tạo từng file cấu hình cho mỗi môi trường riêng biệt
  (`development, test hay production` chẳng hạn),
  và đặt vào các thư mục tương ứng: `config/environments/production`
<sup>[[link](#dev-test-prod-configs)]</sup>

* Đánh dấu những assets để tiền biên dịch (precompilation) (nếu có):

  ```Ruby
  # config/environments/production.rb
  # Precompile additional assets (application.js, application.css,
  #and all non-JS/CSS are already added)
  config.assets.precompile += %w( rails_admin/rails_admin.css rails_admin/rails_admin.js )
  ```

* <a name="app-config"></a>k
  Những config mà dùng chung cho tất cả các môi trường thì đặt trong `config/application.rb`.
<sup>[[link](#app-config)]</sup>

* <a name="staging-like-prod"></a>
  Nên có môi trường `staging` gần như là `production` để phục vụ test trên môi trường thật.
<sup>[[link](#staging-like-prod)]</sup>

* <a name="yaml-config"></a>
  Những config bổ sung thì viết vào các file `YAML` và đặt trong thư mục `config/`.
<sup>[[link](#yaml-config)]</sup>

  Từ Rails 4.2, các file config `YAML` có thể load ra bằng cách :

  ```Ruby
  Rails::Application.config_for(:yaml_file)
  ```

## Routing

* <a name="member-collection-routes"></a>
  Nếu muốn thêm `action` vào `RESTful` resource (cân nhắc kỹ có thực sự cần hay không)
  dùng `member` và `collection` routes.
<sup>[[link](#member-collection-routes)]</sup>

  ```Ruby
  # bad
  get 'subscriptions/:id/unsubscribe'
  resources :subscriptions

  # good
  resources :subscriptions do
    get 'unsubscribe', on: :member
  end

  # bad
  get 'photos/search'
  resources :photos

  # good
  resources :photos do
    get 'search', on: :collection
  end
  ```

* <a name="many-member-collection-routes"></a>
  Nếu cần định nghĩa nhiều `member/collection` routes, dùng block.
<sup>[[link](#many-member-collection-routes)]</sup>

  ```Ruby
  resources :subscriptions do
    member do
      get 'unsubscribe'
      # more routes
    end
  end

  resources :photos do
    collection do
      get 'search'
      # more routes
    end
  end
  ```

* <a name="nested-routes"></a>
  Dùng routes lồng nhau (nested routes) để thể hiện tốt hơn quan hệ
  giữa các ActiveRecord models.
<sup>[[link](#nested-routes)]</sup>

  ```Ruby
  class Post < ActiveRecord::Base
    has_many :comments
  end

  class Comments < ActiveRecord::Base
    belongs_to :post
  end

  # routes.rb
  resources :posts do
    resources :comments
  end
  ```

* <a name="namespaced-routes"></a>
  Nếu lồng nhiều hơn 1 cấp, sử dụng tùy chọn `shallow: true`.
  ```Ruby
  resources :posts, shallow: true do
    resources :comments do
      resources :versions
    end
  end
  ```

* <a name="namespaced-routes"></a>
  Dùng `namespace` routes để nhóm các actions có liên quan.
<sup>[[link](#namespaced-routes)]</sup>

  ```Ruby
  namespace :admin do
    # Directs /admin/products/* to Admin::ProductsController
    # (app/controllers/admin/products_controller.rb)
    resources :products
  end
  ```

* <a name="no-wild-routes"></a>
  Không dùng route thuần cho controller. Nó sẽ làm cho tất cả các `action`
  trong các `controller` có thể truy cập được thông qua `GET` request.
<sup>[[link](#no-wild-routes)]</sup>

  ```Ruby
  # very bad
  match ':controller(/:action(/:id(.:format)))'
  ```

* <a name="no-match-routes"></a>
  Không dùng `match` để khai báo route trừ khi cần thiết để map các action vào với nhau
  `[:get, :post, :patch, :put, :delete]` vào một `action` duy nhất với tùy chọn `:via`.
<sup>[[link](#no-match-routes)]</sup>

## Controllers

* <a name="skinny-controllers"></a>
  Giữ cho `controller` được "mỏng" - chỉ nên lấy dữ liệu cho `View` và
  không nên chứa bất kỳ `business logic` nào.
  `business logic` thì nên để trong `model` hoặc trong `service`
<sup>[[link](#skinny-controllers)]</sup>

* <a name="one-method"></a>
  Ngoài khởi tạo hay tìm kiếm, mỗi `controller action` nên gọi một phương thức.
<sup>[[link](#one-method)]</sup>

* <a name="shared-instance-variables"></a>
  Giữa `controller` và `view` chỉ nên dùng chung tối đa hai biến toàn cục
<sup>[[link](#shared-instance-variables)]</sup>


### Rendering

* <a name="inline-rendering"></a>
  Ưu tiên sử dụng `template` hơn là `inline rendering`
<sup>[[link](#inline-rendering)]</sup>

```Ruby
# very bad
class ProductsController < ApplicationController
  def index
    render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>", type: :erb
  end
end

# good
## app/views/products/index.html.erb
<%= render partial: 'product', collection: products %>

## app/views/products/_product.html.erb
<p><%= product.name %></p>
<p><%= product.price %></p>

## app/controllers/foo_controller.rb
class ProductsController < ApplicationController
  def index
    render :index
  end
end
```

* <a name="plain-text-rendering"></a>
  Ưu tiên `render plain:` hơn `render text:`.
<sup>[[link](#plain-text-rendering)]</sup>

```Ruby
# bad - sets MIME type to `text/html`
...
render text: 'Ruby!'
...

# bad - requires explicit MIME type declaration
...
render text: 'Ruby!', content_type: 'text/plain'
...

# good - short and precise
...
render plain: 'Ruby!'
...
```

* <a name="http-status-code-symbols"></a>
  Ưu tiên [corresponding symbols](https://gist.github.com/mlanett/a31c340b132ddefa9cca) cho HTTP status codes.
  Nó dễ hiểu hơn là dùng số cứng.
<sup>[[link](#http-status-code-symbols)]</sup>

```Ruby
# bad
...
render status: 500
...

# good
...
render status: :forbidden
...
```

## Models

* <a name="model-classes"></a>
  Có thể tạo non-ActiveRecord model thỏa mái.
<sup>[[link](#model-classes)]</sup>

* <a name="meaningful-model-names"></a>
  Tên model nên đặt tên ngắn gọn, có nghĩa và không dùng từ viết tắt.
<sup>[[link](#meaningful-model-names)]</sup>

* <a name="activeattr-gem"></a>
  Nếu cần model vẫn hỗ trợ các hành vi của ActiveRecord (vd: validation)
  mà không có `ActiveRecord database`, dùng [ActiveAttr](https://github.com/cgriego/active_attr) gem.
<sup>[[link](#activeattr-gem)]</sup>

  ```Ruby
  class Message
    include ActiveAttr::Model

    attribute :name
    attribute :email
    attribute :content
    attribute :priority

    attr_accessible :name, :email, :content

    validates :name, presence: true
    validates :email, format: { with: /\A[-a-z0-9_+\.]+\@([-a-z0-9]+\.)+[a-z0-9]{2,4}\z/i }
    validates :content, length: { maximum: 500 }
  end
  ```

  Xem thêm:
  [RailsCast on the subject](http://railscasts.com/episodes/326-activeattr).

* <a name="model-business-logic"></a>
  Unless they have some meaning in the business domain, don't put methods in
  your model that just format your data (like code generating HTML). These
  methods are most likely going to be called from the view layer only, so their
  place is in helpers. Keep your models for business logic and data-persistence
  only.
<sup>[[link](#model-business-logic)]</sup>

### ActiveRecord

* <a name="keep-ar-defaults"></a>
  Tránh thay thế các giá trị mặc định của ActiveRecord (như: tên bảng, khóa chính...)
  trừ khi có lý cho chính đáng.
<sup>[[link](#keep-ar-defaults)]</sup>

  ```Ruby
  # bad - don't do this if you can modify the schema
  class Transaction < ActiveRecord::Base
    self.table_name = 'order'
    ...
  end
  ```

* <a name="macro-style-methods"></a>
  Nhóm các khối phương thưc `macro-style` (`has_many`, `validates`...) vào đầu lớp.
<sup>[[link](#macro-style-methods)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    # keep the default scope first (if any)
    default_scope { where(active: true) }

    # constants come up next
    COLORS = %w(red green blue)

    # afterwards we put attr related macros
    attr_accessor :formatted_date_of_birth

    attr_accessible :login, :first_name, :last_name, :email, :password

    # Rails4+ enums after attr macros, prefer the hash syntax
    enum gender: { female: 0, male: 1 }

    # followed by association macros
    belongs_to :country

    has_many :authentications, dependent: :destroy

    # and validation macros
    validates :email, presence: true
    validates :username, presence: true
    validates :username, uniqueness: { case_sensitive: false }
    validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
    validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true }

    # next we have callbacks
    before_save :cook
    before_save :update_username_lower

    # other macros (like devise's) should be placed after the callbacks

    ...
  end
  ```

* <a name="has-many-through"></a>
  Ưu tiên dùng `has_many :through` hơn `has_and_belongs_to_many`. Dùng `has_many
  :through` cho phép ta có thêm các thuộc tính và các bước xác thực trong `join model`.
<sup>[[link](#has-many-through)]</sup>

  ```Ruby
  # not so good - using has_and_belongs_to_many
  class User < ActiveRecord::Base
    has_and_belongs_to_many :groups
  end

  class Group < ActiveRecord::Base
    has_and_belongs_to_many :users
  end

  # preferred way - using has_many :through
  class User < ActiveRecord::Base
    has_many :memberships
    has_many :groups, through: :memberships
  end

  class Membership < ActiveRecord::Base
    belongs_to :user
    belongs_to :group
  end

  class Group < ActiveRecord::Base
    has_many :memberships
    has_many :users, through: :memberships
  end
  ```

* <a name="read-attribute"></a>
  Ưu tiên `self[:attribute]` hơn `read_attribute(:attribute)`.
<sup>[[link](#read-attribute)]</sup>

  ```Ruby
  # bad
  def amount
    read_attribute(:amount) * 100
  end

  # good
  def amount
    self[:amount] * 100
  end
  ```

* <a name="write-attribute"></a>
  Ưu tiên `self[:attribute] = value` hơn `write_attribute(:attribute, value)`.
<sup>[[link](#write-attribute)]</sup>

  ```Ruby
  # bad
  def amount
    write_attribute(:amount, 100)
  end

  # good
  def amount
    self[:amount] = 100
  end
  ```

* <a name="sexy-validations"></a>
  Luôn dùng ["sexy"
  validations](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/).
<sup>[[link](#sexy-validations)]</sup>

  ```Ruby
  # bad
  validates_presence_of :email
  validates_length_of :email, maximum: 100

  # good
  validates :email, presence: true, length: { maximum: 100 }
  ```

* <a name="custom-validator-file"></a>
  Khi một `custom validation` mà dùng nhiều hơn một `validation` khác,
  hoặc nó là `regex mapping` thì nên tạo một `custom validator file`.
<sup>[[link](#custom-validator-file)]</sup>

  ```Ruby
  # bad
  class Person
    validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
  end

  # good
  class EmailValidator < ActiveModel::EachValidator
    def validate_each(record, attribute, value)
      record.errors[attribute] << (options[:message] || 'is not a valid email') unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
    end
  end

  class Person
    validates :email, email: true
  end
  ```

* <a name="app-validators"></a>
  `custom validators` đặt ở thư mục `app/validators`.
<sup>[[link](#app-validators)]</sup>

* <a name="custom-validators-gem"></a>
  Nếu `custom validators` được dùng đi dùng lại ở nhiều dự án thì nên đưa nó vào
  `gem`.
<sup>[[link](#custom-validators-gem)]</sup>

* <a name="named-scopes"></a>
  Scope được phép dùng thỏa mái.
<sup>[[link](#named-scopes)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    scope :active, -> { where(active: true) }
    scope :inactive, -> { where(active: false) }

    scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
  end
  ```

* <a name="named-scope-class"></a>
  Khi scope quá phức tạp, nên chuyển sang `class method`
  và trả về một `ActiveRecord::Relation`
<sup>[[link](#named-scope-class)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    def self.with_orders
      joins(:orders).select('distinct(users.id)')
    end
  end
  ```

* <a name="beware-update-attribute"></a>
  Cẩn thận với phương thức
  [`update_attribute`](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute).
  Nó không gọi `model validations` (unlike `update_attributes`) và có thể
  sẽ làm lỗi model.
<sup>[[link](#beware-update-attribute)]</sup>

* <a name="user-friendly-urls"></a>
  Dùng URL thân thiên, dễ đọc với người dùng. Thay vì chỉ show mình `id` trên URL,
  hãy show thêm vài thông tin có ích. Có nhiều cách để làm việc này:
<sup>[[link](#user-friendly-urls)]</sup>

  * Override phương thức `to_param` của model. Phương thức này được Rails dùng
    mặc định khi sinh URL cho đối tượng. Mặc định nó trả về `id`.

      ```Ruby
      class Person
        def to_param
          "#{id} #{name}".parameterize
        end
      end
      ```

  Phương thức `parameterize` nên được gọi từ một String. Nên đặt `id`
  ở đầu để `ActiveRecord` có thể dùng được phương thức `find` để gọi nó.

  * Dùng `friendly_id` gem.

      ```Ruby
      class Person
        extend FriendlyId
        friendly_id :name, use: :slugged
      end
      ```

  Xem thêm [gem documentation](https://github.com/norman/friendly_id).

* <a name="find-each"></a>
  Dùng `find_each` để duyệt qua một danh sách các `ActiveRecord`.
  Lặp qua một `collection` các record từ CSDL (vd: dùng phương thức `all`)
  rất kém hiệu quả, nguyên do là mỗi lần nó sẽ tạo ra một đối tượng.
  Thay vào đó, hãy dùng `find_each` để gom chúng vào và làm một lần.
<sup>[[link](#find-each)]</sup>


  ```Ruby
  # bad
  Person.all.each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').each do |person|
    person.party_all_night!
  end

  # good
  Person.find_each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').find_each do |person|
    person.party_all_night!
  end
  ```

* <a name="before_destroy"></a>
  Từ khi [Rails creates callbacks for dependent
  associations](https://github.com/rails/rails/issues/3458) ra đời, luôn gọi
  `before_destroy` callbacks cùng với `prepend: true` để validate.
<sup>[[link](#before_destroy)]</sup>

  ```Ruby
  # bad (roles will be deleted automatically even if super_admin? is true)
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end

  # good
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable, prepend: true

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end
  ```

* <a name="has_many-has_one-dependent-option"></a>
  Định nghĩa tùy chọn `dependent` cho quan hệ `has_many` và `has_one`.
<sup>[[link](#has_many-has_one-dependent-option)]</sup>

  ```Ruby
  # bad
  class Post < ActiveRecord::Base
    has_many :comments
  end

  # good
  class Post < ActiveRecord::Base
    has_many :comments, dependent: :destroy
  end
  ```

### ActiveRecord Queries

* <a name="avoid-interpolation"></a>
  Tránh nhúng biến vào String của câu truy vấn, nó sẽ dễ dẫn đến SQL injection.
<sup>[[link](#avoid-interpolation)]</sup>

  ```Ruby
  # bad - param will be interpolated unescaped
  Client.where("orders_count = #{params[:orders]}")

  # good - param will be properly escaped
  Client.where('orders_count = ?', params[:orders])
  ```

* <a name="named-placeholder"></a>
  Nếu câu truy vấn có nhiều hơn một tham số, ưu tiên dùng phương án
  tham-số-là-tên (named placeholders) thay vì dùng `?`.
  Việc này sẽ giúp bạn tránh việc nhầm lẫn, sai sót khi các tham số không
  được đặt đúng thứ tự.
<sup>[[link](#named-placeholder)]</sup>

  ```Ruby
  # okish
  Client.where(
    'created_at >= ? AND created_at <= ?',
    params[:start_date], params[:end_date]
  )

  # good
  Client.where(
    'created_at >= :start_date AND created_at <= :end_date',
    start_date: params[:start_date], end_date: params[:end_date]
  )
  ```

* <a name="find"></a>
  Khi cần query một record bằng `id`, ưu tiên dùng `find` hơn `where`
<sup>[[link](#find)]</sup>

  ```Ruby
  # bad
  User.where(id: id).take

  # good
  User.find(id)
  ```

* <a name="find_by"></a>
  Khi cần query một record bằng một vài attribute, ưu tiên dùng `find_by`
  hơn `where` và `find_by_attribute`.
<sup>[[link](#find_by)]</sup>

  ```Ruby
  # bad
  User.where(first_name: 'Bruce', last_name: 'Wayne').first

  # bad
  User.find_by_first_name_and_last_name('Bruce', 'Wayne')

  # good
  User.find_by(first_name: 'Bruce', last_name: 'Wayne')
  ```

* <a name="where-not"></a>
  Ưu tiên dùng `where.not` hơn là SQL thuần.
<sup>[[link](#where-not)]</sup>

  ```Ruby
  # bad
  User.where("id != ?", id)

  # good
  User.where.not(id: id)
  ```
* <a name="squished-heredocs"></a>
  Khi dùng query trực tiếp bằng `find_by_sql`, dùng heredocs cùng với `squish`.
  Bạn sẽ format code theo SQL tốt hơn (xuống dòng, thụt đầu dòng, tô màu từ khóa)
  và được hiển thị tốt ở nhiều tools như GitHub, Atom, and RubyMine.
<sup>[[link](#squished-heredocs)]</sup>

  ```Ruby
  User.find_by_sql(<<SQL.squish)
    SELECT
      users.id, accounts.plan
    FROM
      users
    INNER JOIN
      accounts
    ON
      accounts.user_id = users.id
    # further complexities...
  SQL
  ```

  [`String#squish`](http://apidock.com/rails/String/squish) sẽ tự bỏ thụt đầu dòng
  hay ký hiệu xuống dòng, nên khi log ra sẽ rất khó đọc, kiểu như này:
  ```
  SELECT\n    users.id, accounts.plan\n  FROM\n    users\n  INNER JOIN\n    acounts\n  ON\n    accounts.user_id = users.id
  ```

## Migrations

* <a name="schema-version"></a>
  Đưa `schema.rb` (hoặc `structure.sql`) vào version control system (git, hg...).
<sup>[[link](#schema-version)]</sup>

* <a name="db-schema-load"></a>
  Dùng `rake db:schema:load` thay vì `rake db:migrate` để khởi tạo CSDL rỗng.
<sup>[[link](#db-schema-load)]</sup>

* <a name="default-migration-values"></a>
  Khi cần tạo dữ liệu mặc định, ưu tiên tạo trong migration hơn là trong
  code (tầng application).
<sup>[[link](#default-migration-values)]</sup>

  ```Ruby
  # bad - application enforced default value
  def amount
    self[:amount] or 0
  end
  ```

  Mặc dù rất nhiều Rails dev khuyến nghị tạo dữ liệu mặc định trong code,
  tuy nhiên, điều này rất phiêu. Ví dụ như trường hợp nhiều app nhỏ nhỏ thường
  dùng chung một CSDL, sẽ dẫn đến việc mất tính toàn vẹn dữ liệu.

* <a name="foreign-key-constraints"></a>
  Dùng khóa ngoại (`foreign-key constraints`) để đảm bảo tính toàn vẹn dữ liệu.
  Ở Rails 4.2, ActiveRecord đã hỗ trợ foreign key constraints natively.
  <sup>[[link](#foreign-key-constraints)]</sup>

* <a name="change-vs-up-down"></a>
  Khi viết các `migration` kiểu khởi tạo `bảng`, thêm `cột`, dùng `change`
  thay vì `up` và `down`.
  Bởi vì trong trường hợp này thì `Rails` nó tự biết để `revert, rollback` rồi.
  <sup>[[link](#change-vs-up-down)]</sup>

  ```Ruby
  # the old way
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # the new preferred way
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```

* <a name="no-model-class-migrations"></a>
  Không dùng `model` trong migration. Vì `model` có thể sẽ thay đổi trong
  tương lai (đổi tên, xóa phương thức...)
<sup>[[link](#no-model-class-migrations)]</sup>

## Views

* <a name="no-direct-model-view"></a>
  Không gọi `model` trực tiếp trong `view`.
<sup>[[link](#no-direct-model-view)]</sup>

* <a name="no-complex-view-formatting"></a>
  Không xử lý logic phức tạp trong `view`, chuyển nó sang `model` hoặc `helper`.
<sup>[[link](#no-complex-view-formatting)]</sup>

* <a name="partials"></a>
  Dùng `partial` để tránh lặp lại code (DRY).
<sup>[[link](#partials)]</sup>

## Internationalization (I18n)

* <a name="locale-texts"></a>
  Không nên đặt các `string`, `setting` trong `model`, `view` hay `controller`.
  Nên dùng `I18n` để trong các file config và đặt ở thư mục `config/locales`.
<sup>[[link](#locale-texts)]</sup>

* <a name="translated-labels"></a>
  Dùng `activerecord` scope để dịch các `label` của ActiveRecord.
<sup>[[link](#translated-labels)]</sup>

  ```
  en:
    activerecord:
      models:
        user: Member
      attributes:
        user:
          name: 'Full name'
  ```

  Gọi `User.model_name.human` sẽ trả về "Member" và
  `User.human_attribute_name("name")` sẽ trả về "Full name".


* <a name="organize-locale-files"></a>
  Sử dụng hai thư mục `locales/models` và `locales/views` để đặt các file
  tương ứng.
<sup>[[link](#organize-locale-files)]</sup>

  * Lưu ý là thư mục `locales` cần được khai báo trong `application.rb` để nó được
    load khi chạy app.

      ```Ruby
      # config/application.rb
      config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
      ```

* <a name="shared-localization"></a>
  Những option dùng chung, ví dụ như ngày tháng hay đơn vị...trong thư mục gốc
  của `locales`.
<sup>[[link](#shared-localization)]</sup>

* <a name="short-i18n"></a>
  Dùng `I18n.t` thay cho `I18n.translate` và `I18n.l` thay cho `I18n.localize`.
<sup>[[link](#short-i18n)]</sup>

* <a name="lazy-lookup"></a>
  Dùng `"lazy" lookup` cho text dùng trong `views`. Ví dụ ta có cấu trúc sau:
<sup>[[link](#lazy-lookup)]</sup>

  ```
  en:
    users:
      show:
        title: 'User details page'
  ```

  Trong file `app/views/users/show.html.haml`, thay vì gọi `users.show.title`
  ta có thể dùng:

  ```Ruby
  = t '.title'
  ```

* <a name="dot-separated-keys"></a>
  Dùng kiểu phân cấp bằng dấu chấm thay vì dùng `scope`, dễ đọc hơn.
<sup>[[link](#dot-separated-keys)]</sup>

  ```Ruby
  # bad
  I18n.t :record_invalid, scope: [:activerecord, :errors, :messages]

  # good
  I18n.t 'activerecord.errors.messages.record_invalid'
  ```

* <a name="i18n-guides"></a>
  Tài liệu đầy đủ về [I18n](http://guides.rubyonrails.org/i18n.html)
<sup>[[link](#i18n-guides)]</sup>

## Assets

Dùng [assets pipeline](http://guides.rubyonrails.org/asset_pipeline.html)
để tổ chức `assets`.

* <a name="reserve-app-assets"></a>
  Để giành thư mục `app/assets` cho các custom stylesheets, javascripts, hay images.
<sup>[[link](#reserve-app-assets)]</sup>

* <a name="lib-assets"></a>
  Dùng thư mục `lib/assets` cho các `assets` của thư viện.
<sup>[[link](#lib-assets)]</sup>

* <a name="vendor-assets"></a>
  Các thư viện bên thứ ba như [jQuery](http://jquery.com/) hay
  [bootstrap](http://twitter.github.com/bootstrap/) thì đặt ở thư mục
  `vendor/assets`.
<sup>[[link](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  Dùng `gemified versions` của assets khi có thể (vd:
  [jquery-rails](https://github.com/rails/jquery-rails),
  [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails),
  [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass),
  [zurb-foundation](https://github.com/zurb/foundation)).
<sup>[[link](#gem-assets)]</sup>

## Mailers

* <a name="mailer-name"></a>
  Đặt tên `SomethingMailer` sẽ giúp rõ nghĩa hơn về chức năng của nó, và `view`
  nào sẽ được sử dụng.
<sup>[[link](#mailer-name)]</sup>

* <a name="html-plain-email"></a>
  Dùng cả `HTML` và `plain-text` trong `view templates`.
<sup>[[link](#html-plain-email)]</sup>

* <a name="enable-delivery-errors"></a>
  Bật tùy chọn thông báo lỗi khi gửi email thất bại, dùng trong môi trường
  `development`, mặc định là nó bị tắt.
<sup>[[link](#enable-delivery-errors)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  Dùng một `local SMTP server`, chẳng hạn
  [Mailcatcher](https://github.com/sj26/mailcatcher) trong môi trường `development`.
<sup>[[link](#local-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.smtp_settings = {
    address: 'localhost',
    port: 1025,
    # more settings
  }
  ```

* <a name="default-hostname"></a>
  Cài đặt các thông số mặc định cho host name.
<sup>[[link](#default-hostname)]</sup>

  ```Ruby
  # config/environments/development.rb
  config.action_mailer.default_url_options = { host: "#{local_ip}:3000" }

  # config/environments/production.rb
  config.action_mailer.default_url_options = { host: 'your_site.com' }

  # in your mailer class
  default_url_options[:host] = 'your_site.com'
  ```

* <a name="url-not-path-in-email"></a>
  Nếu muốn nhúng `link` của server vào `email`, Sử dụng phương thức `_url` thay vì
  `_path`.
<sup>[[link](#url-not-path-in-email)]</sup>

  ```Ruby
  # bad
  You can always find more info about this course
  <%= link_to 'here', course_path(@course) %>

  # good
  You can always find more info about this course
  <%= link_to 'here', course_url(@course) %>
  ```

* <a name="email-addresses"></a>
  Format `from` và `addresses` theo mẫu:
<sup>[[link](#email-addresses)]</sup>

  ```Ruby
  # in your mailer class
  default from: 'Your Name <info@your_site.com>'
  ```

* <a name="delivery-method-test"></a>
  Cấu hình cho môi trường `test`:
<sup>[[link](#delivery-method-test)]</sup>

  ```Ruby
  # config/environments/test.rb

  config.action_mailer.delivery_method = :test
  ```

* <a name="delivery-method-smtp"></a>
  Cấu hình cho môi trường `development` và `production`, dùng `smtp`:
<sup>[[link](#delivery-method-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb, config/environments/production.rb

  config.action_mailer.delivery_method = :smtp
  ```

* <a name="inline-email-styles"></a>
  Khi gửi `html email`, các `style` nên làm `inline`, vì một vài `mail client`
  không hỗ trợ việc dùng các `style` ngoài. Việc này sẽ dẫn đến vấn đề là code
  rất khó bảo trì, và bị lặp code rất nhiều. Có 2 gem hỗ trợ khắc phục vấn đề này:
  [premailer-rails](https://github.com/fphilipe/premailer-rails) và
  [roadie](https://github.com/Mange/roadie).
<sup>[[link](#inline-email-styles)]</sup>

* <a name="background-email"></a>
  Nên gửi email trong `background process`, hoặc dùng
  [sidekiq](https://github.com/mperham/sidekiq) gem.
<sup>[[link](#background-email)]</sup>


## Active Support Core Extensions

* <a name="try-bang"></a>
  Ưu tiên dùng toán tử truy xuất an toàn của Ruby 2.3 `&.` hơn `ActiveSupport#try!`.
<sup>[[link](#try-bang)]</sup>

```ruby
# bad
obj.try! :fly

# good
obj&.fly
```

* <a name="active_support_aliases"></a>
  Ưu tiên dùng các hàm trong thư viện chuẩn của Ruby hơn
  các bí danh của `ActiveSupport`.
<sup>[[link](#active_support_aliases)]</sup>

```ruby
# bad
'the day'.starts_with? 'th'
'the day'.ends_with? 'ay'

# good
'the day'.start_with? 'th'
'the day'.end_with? 'ay'
```

* <a name="active_support_extensions"></a>
  Ưu tiên dùng thư viện chuẩn của Ruby hơn các extension của  `ActiveSupport`
  không thông dụng.
<sup>[[link](#active_support_extensions)]</sup>

```ruby
# bad
(1..50).to_a.forty_two
1.in? [1, 2]
'day'.in? 'the day'

# good
(1..50).to_a[41]
[1, 2].include? 1
'the day'.include? 'day'
```

* <a name="inquiry"></a>
  Ưu tiên dùng toán tử so sánh của `ActiveSupport` hơn `Array#inquiry`,
  `Numeric#inquiry` và `String#inquiry`.
<sup>[[link](#inquiry)]</sup>

```ruby
# bad - String#inquiry
ruby = 'two'.inquiry
ruby.two?

# good
ruby = 'two'
ruby == 'two'

# bad - Array#inquiry
pets = %w(cat dog).inquiry
pets.gopher?

# good
pets = %w(cat dog)
pets.include? 'cat'

# bad - Numeric#inquiry
0.positive?
0.negative?

# good
0 > 0
0 < 0
```

## Time

* <a name="tz-config"></a>
  Cấu hình `timezone` trong `application.rb`.
<sup>[[link](#tz-config)]</sup>

  ```Ruby
  config.time_zone = 'Eastern European Time'
  # optional - note it can be only :utc or :local (default is :utc)
  config.active_record.default_timezone = :local
  ```

* <a name="time-parse"></a>
  Không dùng `Time.parse`, thay vào đó hãy dùng `Time.zone.parse`
  `Time.parse` sẽ lấy `timezone` của hệ thống.
<sup>[[link](#time-parse)]</sup>

  ```Ruby
  # bad
  Time.parse('2015-03-02 19:05:37')

  # good
  Time.zone.parse('2015-03-02 19:05:37') # => Mon, 02 Mar 2015 19:05:37 EET +02:00
  ```

* <a name="time-now"></a>
  Không dùng `Time.now`.
  `Time.now` sẽ lấy `timezone` của hệ thống.
<sup>[[link](#time-now)]</sup>

  ```Ruby
  # bad
  Time.now

  # good
  Time.zone.now # => Fri, 12 Mar 2014 22:04:47 EET +02:00
  Time.current # Tương tự nhưng viết ngắn hơn
  ```

## Bundler

* <a name="dev-test-gems"></a>
  Nếu gem chỉ dùng trong môi trường `development` hay `testing`, đặt chúng vào
  scope tương ứng.
<sup>[[link](#dev-test-gems)]</sup>

* <a name="os-specific-gemfile-locks"></a>
  Có một số gem sẽ có phiên bản khác nhau cho mỗi hệ điều hành khác nhau,
  trường hợp này thì đặt chúng vào `group` tương ứng:
  `darwin` cho OS X và `linux` cho Linux:
<sup>[[link](#os-specific-gemfile-locks)]</sup>

  ```Ruby
  # Gemfile
  group :darwin do
    gem 'rb-fsevent'
    gem 'growl'
  end

  group :linux do
    gem 'rb-inotify'
  end
  ```

  Để yêu cầu lập trình viên phải dev trên hệ điều hành nào đó, cấu hình nó trong
  file `config/application.rb`:

  ```Ruby
  platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
  Bundler.require(platform)
  ```

* <a name="gemfile-lock"></a>
  Thêm file `Gemfile.lock` vào version control system (git, hg). Nó sẽ đảm bảo
  các thành viên trong team dùng chung một phiên bản của các `gem`.
<sup>[[link](#gemfile-lock)]</sup>

## Managing processes

* <a name="foreman"></a>
  Nếu dự án chạy nhiều process, dùng
  [foreman](https://github.com/ddollar/foreman) để quản lý.
<sup>[[link](#foreman)]</sup>

# Further Reading

Dưới đây là những `resources` cũng rất đáng quan tâm:

* [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)
* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)
* [Better Specs for RSpec](http://betterspecs.org)

# Contributing

Nothing written in this guide is set in stone. It's my desire to work together
with everyone interested in Rails coding style, so that we could ultimately
create a resource that will be beneficial to the entire Ruby community.

Feel free to open tickets or send pull requests with improvements. Thanks in
advance for your help!

You can also support the project (and RuboCop) with financial contributions via
[gittip](https://www.gittip.com/bbatsov).

[![Support via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)

## How to Contribute?

It's easy, just follow the [contribution guidelines](https://github.com/bbatsov/rails-style-guide/blob/master/CONTRIBUTING.md).

# License

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 3.0 Unported
License](http://creativecommons.org/licenses/by/3.0/deed.en_US)

# Spread the Word

A community-driven style guide is of little use to a community that doesn't know
about its existence. Tweet about the guide, share it with your friends and
colleagues. Every comment, suggestion or opinion we get makes the guide just a
little bit better. And we want to have the best possible guide, don't we?

Cheers,<br/>
[Bozhidar](https://twitter.com/bbatsov)
