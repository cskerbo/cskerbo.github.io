---
layout: post
title:      "Connecting to Philips Hue API in Rails"
date:       2020-03-14 22:09:15 +0000
permalink:  connecting_to_philips_hue_api_in_rails
---

I recently completed a home dashboard application called HomeBoard for a Rails Portfolio Project. One of the features I wanted to include in the dashboard was the ability for users to interact with their Philips Hue light system. Surprisingly I didn't see a lot of Ruby-specific examples of this out in the wild, so I hope that this information proves useful to someone else out there as I walk through it.

**About HomeBoard**

<a href="https://imgur.com/DxGK4PL"><img src="https://i.imgur.com/DxGK4PL.png" title="source: imgur.com" style="width: 450px; height: 300px" /></a>

HomeBoard is a Content Management System designed for your home. Designed for busy families who need quick access to shared information within the house. An ideal usage for HomeBoard is to display it on a touchscreen monitor within the most active area of your home.

* Track and log important daily tasks such as feeding the pets or taking the dog for a walk.
* Add items to a Grocery or To-Do list where everyone can see it.
* Control your Philips Hue light system
* View a shared calendar and add events as needed.
* View up-to-date local weather information
* Display the time, date, and a quote of the day

You can view the HomeBoard repository [here](https://github.com/cskerbo/HomeBoard).

**Philips Hue API**

Philips has great documentation [for their API](https://developers.meethue.com/#), and I was able to locate a few custom APIs that Ruby developers had created - but nothing that had been updated within the last two years.

In my specific situation - I needed the ability to locate a user's Hue Bridge automatically on their home network; then automate the creation of any bulbs, groups, and scenes they had in their home.

Let's walk through how I tackled this.

*I'm providing this tutorial with the assumption that you have experience in Rails, already have an application in need of Hue data, and just might need some help tying the final pieces together.*

## ActiveRecord Associations

* The first model I created was a Bridge model. The bridge belongs to the home, has many bulbs, has many groups, and has many scenes (through groups).

```
class Bridge < ApplicationRecord
  belongs_to :home
  has_many :bulbs
  has_many :groups
  has_many :scenes, through: :groups
  validates_presence_of :identifier, :internalip
end
```

* Next I created the Bulbs model. Bulbs belong to the bridge and the group.

```
class Bulb < ApplicationRecord
  belongs_to :bridge
  belongs_to :group
end
```

* The Groups model belongs to the bridge, has many bulbs, and has many scenes.

```
class Group < ApplicationRecord
  belongs_to :bridge
  has_many :bulbs
  has_many :scenes
end
```

* Lastly, the Scenes model. Scenes belong to the group and the bridge.

```
class Scene < ApplicationRecord
  belongs_to :group
  belongs_to :bridge
end
```

## Finding and Creating Bridge Data

Before you tackle the creation of any bulbs, groups, or scenes - a connection the Hue Bridge must be established. 

**Locate the Bridge**

If a user has a Hue Bridge, it should be discoverable on their network using the "MeetHue" addres referenced in the Hue API. The first method I created was the *find_bridges* method to locate the bridge and store the bridge identifier and IP address.

```
def find_bridges
    bridge_data = {}
    http = Net::HTTP.new('www.meethue.com',443)
    http.use_ssl = true
    request = Net::HTTP::Get.new('/api/nupnp')
    response = http.request request
    case response.code.to_i
    when 200
      result = JSON.parse( response.body )
      bridge_data[:identifier] = result[0]['id']
      bridge_data[:internalip] = result[0]['internalipaddress']
      bridge_data
    else
      raise 'Unknown error'
    end
  end
```

The first step is complete, you now have a method that will search for a Hue Bridge on the user's home network!

**Create the Bridge**

```
def create_bridges(id)
    @bridge = Bridge.create(find_bridges)
 end
```

Create a new bridge object using the hash returned from *find_bridges*.

**Obtain a Username**

Now that we have a bridge object with an IP address saved, we can obtain a username.

```
def register_hue_user(internalip)
    if @bridge.username.nil?
      http = Net::HTTP.new(internalip, 80)
      data = { 'devicetype'=>'homeboard' }
      response = http.post '/api', data.to_json
      result = JSON.parse(response.body).first
      if result.has_key? 'error'
        flash[:error] = result['error']['description']
      elsif result['success']
        @bridge.username = result['success']['username']
        @bridge.save!
        flash[:notice] = 'Bridge connection successful!'
      end
    end
  end
```

If you read through the Hue Documentation, you know that you must connect to the bridge and receive a permanent username to make API calls back and forth.

But wait - connecting to the bridge and obtaining a username requires that a user hit the "Link Button" on their bridge (hitting the link button will allow you to request a username for the next *30 seconds*). How exactly do you work a user-required interaction into your programming logic? .

>  For this step, it's important you think through the user experience. Determine when to enact each step of this processin order to make the experience seamless for the user. How you implement this in your project might vary from this tutorial.


In my case, I wanted a user to be able to specify if they wanted a Light Widget on their dashboard during the initial setup process. In HomeBoard, if a user does specify they want a light widget, I call on the *find_bridges* and *create_bridges* method upon creation of the dashboard.

Once the dashboard is created, the user is prompted to complete bridge setup:

<a href="https://imgur.com/j1ItAN8"><img src="https://i.imgur.com/j1ItAN8.png" title="source: imgur.com" style="width: 500px; height: 300px" /></a>

When the user navigates to the bridge setup page, they are provided with instructions to hit the link button on their bridge. They then have 30 seconds to hit connect.

<a href="https://imgur.com/zLRGKhY"><img src="https://i.imgur.com/zLRGKhY.png" title="source: imgur.com" style="width: 400px; height: 450px" /></a>

When the user hits connect, I am calling on the *register_hue_user* method. A successful connection results in the bridge object being assigned a username. 

The bridge setup is now complete! We should be able to make calls to the Hue API now.

**What's Next?**

Once you have your bridge username and connect to the API, the rest is quite straightforward if you refer to the Hue documentation. 

Using the same logic of "finding a bridge" and "creating a bridge", you can carry this over to bulbs, groups, and scenes. All of this information can be obtained using the Hue API. 

* Find bulbs. Create bulbs using the returned hash.
* Find groups. Create groups using the returned hash.
* Find scenes. Create scenes using the returned hash.

I chose to enact this logic once a successful connection is established to the bridge (basically, once a username is assigned).

In HomeBoard, when a user hits connect and the connection is successful - all bulbs, groups, and scenes are found and created for the user automatically:

<a href="https://imgur.com/fhKLYV8"><img src="https://i.imgur.com/fhKLYV8.png" title="source: imgur.com" style="width: 450px; height: 450px" /></a>

And since each of these is an object stored in a database, you can create a handy widget that allows you to pass each object's data to the Hue API - resulting in light control!

<a href="https://imgur.com/LjBukRk"><img src="https://i.imgur.com/LjBukRk.png" title="source: imgur.com" style="width: 300px; height: 300px" /></a>

By this point, you should have a solid idea how to connect your application to the Philips Hue API. I don't know about you, but being able to control lights through code makes me feel like a wizard.

<a href="https://imgur.com/n6hcwx6"><img src="https://i.imgur.com/n6hcwx6.gif" title="source: imgur.com" /></a>




