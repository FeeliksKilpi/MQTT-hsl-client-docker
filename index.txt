const mqtt = require('mqtt')
const topic = '/hfp/v2/journey/ongoing/vp/bus/+/+/+/+/+/+/+/+/60;24/18/69/27/#';
const HSLclient  = mqtt.connect('mqtts://mqtt.hsl.fi:8883');
const mosquittoClient = mqtt.connect('mqtt://test.mosquitto.org:1883');
const publishTopic = '/swd4tn023/FeeliksKilpi/traffic/jam';

// /<prefix>/<version>/<journey_type>/<temporal_type>/<event_type>/<transport_mode>/<operator_id>/<vehicle_number>/<route_id>/<direction_id>/<headsign>/<start_time>/<next_stop>/<geohash_level>/<geohash>/<sid>/#
// /hfp/v2/journey/ongoing/vp/bus/+/+/+/+/+/+/+/+/60;24/18/69/27/#
// /hfp/v2/journey/ongoing/vp/tram/+/+/+/+/+/+/+/+/60;24/19/62/00/#
// mqtt subscribe -h mqtt.hsl.fi -p 8883 -l mqtts -v -t "/hfp/v2/journey/ongoing/vp/tram/+/+/+/+/+/+/+/+/60;24/19/62/00/#"

HSLclient.on('connect', function () {
  HSLclient.subscribe(topic, function (err) {
    if (!err) {
        console.log('Connection established to HSL client!');
    } else {
        console.log(err);
    }
  })
});

mosquittoClient.on('connect', function () {
    mosquittoClient.subscribe(topic, function (err) {
      if (!err) {
          console.log('Connection established to mosquitto client!');
      } else {
          console.log(err);
      }
    })
  });
 
HSLclient.on('message', function (topic, message) {
  // message is Buffer
  let json = JSON.parse(message.toString());
  let speed = json.VP.spd * 3.6;
  console.log('Bussi ' + json.VP.desi + ', kulkee lauttasaaren sillan yli nopeudella: ' + speed + ' km/h!');
  if (speed < 5.0) {
    let msg = {
        oper: json.VP.oper,
        veh: json.VP.veh,
        lat: json.VP.lat,
        long: json.VP.long,
        spd: json.VP.spd,
        cause: "Potential Traffic Jam",
    }
    let myMessage = speed;
    mosquittoClient.publish('/swd4tn023/FeeliksKilpi/traffic/jam', JSON.stringify(msg));
  }
  else if (speed > 8.33) {
    let msg = {
        oper: json.VP.oper,
        veh: json.VP.veh,
        lat: json.VP.lat,
        long: json.VP.long,
        spd: json.VP.spd,
        cause: "Potential Speeding!",
    }
      let myMessage = speed;
      mosquittoClient.publish('/swd4tn023/FeeliksKilpi/traffic/jam', JSON.stringify(msg));
  }
  HSLclient.end()
})

mosquittoClient.on('message', function (topic, message) {

    mosquittoClient.publish('/swd4tn023/FeeliksKilpi/traffic/jam', JSON.stringify(myMessage));
})



// Test publishing with mqtt cli with following command in teminal:
// mqtt subscribe -h test.mosquitto.org -p 1883 -l mqtt -v -t "/swd4tn023/FeeliksKilpi/traffic/#" 