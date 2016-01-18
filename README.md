## THIS REPOSITORY IS NO LONGER MAINTAINED

New repository home: <https://github.com/PredicSis/swift-ruby>


# Swift::Ruby

Ruby client for Openstack swift

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'swift-ruby'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install swift-ruby

## Usage

```rb
require 'swift-storage'

# = Configuration =
## With environment variables
#
# SWIFT_STORAGE_AUTH_VERSION
# SWIFT_STORAGE_TENANT
# SWIFT_STORAGE_USERNAME
# SWIFT_STORAGE_PASSWORD
# SWIFT_STORAGE_ENDPOINT
# SWIFT_STORAGE_TEMP_URL_KEY
#
## With Rails initializer
#
# Some parameters are by default configured with the above environment variables, see configuration.rb
SwiftStorage.configure do |config|
  config.auth_version = '2.0' # Keystone auth version, default is 1.0
  config.tenant = 'Millennium Falcon' # Aka Openstack project
  config.username = 'han'
  config.password = 'YT-1300'
  config.endpoint = 'https//corellia.lan' # Keystone endpoint
  config.temp_url_key = '492727ZED' # Secret key for presigned URLs
  # ...
end
#
## With service initialization
#
# NB: It overrides initializer configuration
swift = SwiftStorage::Service.new(
  tenant: 'Millennium Falcon',
  username: 'han',
  password: 'YT-1300',
  endpoint: 'https//corellia.lan',
  temp_url_key: '492727ZED'
)


# Authenticate, primary to retrieve Swift Storage URL
swift.authenticate!
swift.authenticated?
# => true

# Setup Secret key in Swift server
swift.account.write(temp_url_key: '492727ZED')

# Create & get containers
swift.containers['source'].create unless swift.containers['source'].exists?
source = swift.containers['source']
swift.containers['destination'].create unless swift.containers['destination'].exists?
destination = swift.containers['destination']

# Get objects
source_obj = source.objects['Kessel.asteroid']
destination_obj = destination.objects['SiKlaata.cluster']

# Upload data into object
source_obj.write('Glitterstim', content_type: 'application/spice')
#  or stream from file
File.open('/tmp/Kessel.asteroid', 'r') do |input|
  source_obj.write(input, content_type: 'application/spice')
end

# Upload data larger than 5GB
# 1/ Split file larger than 5GB into subfiles, for example:
system("split -a 5 -d -b size file_path result_path")
# 2/ upload each part:
File.open(result_path) do |input|
  source_obj.write(input, content_type: 'application/txt', part_location: 'container/object/part_number')
end
# 3/ Create a manifest
source_obj.write(object_manifest: 'container/object/')

# Copy an object
#   Source can be a SwiftStorage::Object or a string like 'source/Kessel.asteroid'
destination_obj.copy_from(source_obj)

# Read data from Swift
p destination_obj.read
# => Glitterstim

# Download to a file
File.open('/tmp/SiKlaata.cluster', 'w') do |output|
  destination_obj.read(output)
end
#  or
destination_obj.stream_to_file('/tmp/SiKlaata.cluster')

# Create temporary pre-signed URL
p destination_obj.temp_url(Time.now + (3600 * 10), method: :get)
# => https//corellia.lan/v1/AUTH_39c47bfd3ecd41938368239813628963/destination/death/star.moon?temp_url_sig=cbd7568b60abcd5862a96eb03af5fa154e851d54&temp_url_expires=1439430168
```

## Contributing

1. Fork it ( https://github.com/[my-github-username]/swift-ruby/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request


## Contributors

- Nicolas Goy @kuon
- @mdouchement
- @ArmandPredicSis
