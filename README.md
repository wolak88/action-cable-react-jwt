# action-cable-react-with-jwt

Original package is no longer supported. This is updated fork which uses actioncable in version 6.1

Same as [action-cable-react](https://github.com/schneidmaster/action-cable-react), but allows authenticating websockets using JWTs

## Installation

Yarn:

```javascript
yarn add actioncable-js-jwt

```

npm:

```javascript
npm install actioncable-js-jwt

```

## Usage

Creating an actioncable websocket

```javascript
import { createConsumer } from '@rails/actioncable';

let App = {};
App.cable = createConsumer("ws://cable.example.com", jwt); // place your jwt here

// you shall also use this.cable = ActionCable.createConsumer(...)
// to create the connection as soon as the view loads, place this in componentDidMount
```

Subscribing to a Channel for Receiving data

```javascript
this.subscription = App.cable.subscriptions.create({channel: "YourChannel"}, {
      connected: function() { console.log("cable: connected") },             // onConnect
      disconnected: function() { console.log("cable: disconnected") },       // onDisconnect
      received: (data) => { console.log("cable received: ", data); }         // OnReceive
}

```

Send data to a channel

```javascript
this.subscription.send("hello world");
```

Call a method on channel with arguments

```javascript
this.subscription.perform("method_name", arguments);
```

In your `ApplicationCable::Connection` class in Ruby add

```ruby
# app/channels/application_cable/connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private

    def find_verified_user
      begin
        header_array = request.headers[:HTTP_SEC_WEBSOCKET_PROTOCOL].split(',')
        token = header_array[header_array.length-1]
        decoded_token = JWT.decode token.strip, Rails.application.secrets.secret_key_base, true, { :algorithm => 'HS256' }
        if (current_user = User.find((decoded_token[0])['sub']))
          current_user
        else
          reject_unauthorized_connection
        end
      rescue
        reject_unauthorized_connection
      end
    end

  end
end
```

And in YourChannel.rb

```ruby
# app/channels/you_channel.rb
class LocationChannel < ApplicationCable::Channel

  # calls connect in client
  def subscribed
    stream_from 'location_user_' + current_user.id.to_s
  end

  # calls disconnect in client
  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end

  # called when send is called in client
  def receive(params)
    print params[:data]
  end

  # called when perform is called in client
  def method_name(params)
    print params[:data]
  end

end
```

Remove a subscription from cable

```javascript
App.cable.subscriptions.remove(this.subscription);

// Place this in componentWillUnmount to remove subscription on exiting app
```

Add a subscription to cable

```javascript
App.cable.subscriptions.add(this.subscription);
```

Querying url and jwt from cable

```javascript
console.log(App.cable.jwt);
console.log(App.cable.url);
```

Querying subscriptions and connection from cable

```javascript
console.log(App.cable.subscriptions);
console.log(App.cable.connection);
```