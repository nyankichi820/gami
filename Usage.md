#Golang Asterisk Manager Interface library.

# gami #

gami is simple library to interact with Asterisk PBX with it manager interface.


# Usage #

Check example in `gami/example/main.go`

## Installation ##

```
go get code.google.com/p/gami
```

in your imports

```
import (
 gami "code.google.com/p/gami"
)
```

## Login/Logoff to AMI ##

```
c, err := net.Dial("tcp", "asterisk_host:5038")
checkErr(err)
...
g = gami.NewAsterisk(&c, nil) // with 2nd parameter you can pass callback function which will execute on network error, func(error)
err = g.Login("user", "passwd") // login call will block execution and return error on fail
...
g.Logoff()
```

## Run simple command ##

```
m := gami.Message{"Action":"Ping"} // gami.Message simple alias on map[string]string
callback := function(m gami.Message) { // most 'gami' commands receive callback function which will execute with command response message as argument, after executing callback will be deleted
 fmt.Println(m)
}
g.SendAction(m, &callback)
```

## Default handler ##

You can register default handler for each event received from AMI, example

```
debug_chan := make(chan gami.Message, 100)
cnt := 0
dh := function(m gami.Message) {
 debug_chan <- m
 cnt++
 fmt.Printf("Received %d events\n", cnt)
}

g.DefaultHandler(&dh)

go handleDebug(debug_chan)
```

You can remove it
```
g.DefaultHandler(nil)
```

## Placing a call ##

**call to asterisk application
```
o := gami.NewOriginateApp("SIP/1234", "playback", "hello-world") // after pick up will playback hello-world to callee
vars := map[string]string{"var1":"val1", "var2":"val2"} // you can pass channels vars with map
respch := make(chan bool) // will wait response by this channel
cb := function(m gami.Message) { // will be executed after call pick up/reject/error
 if m["Response"] == "Success" {
  respch <- true
 } else {
  respch <- false
 }
}
err := g.Originate(o, vars, &cb)
if <- respch {
 fmt.Println("Call placed!")
} else {
 fmt.Println("Can't call")
}
```**

**call to asterisk dialplan
```
// will move answered channel to context 'demo' in default extension
o := gami.NewOriginate("SIP/1234", "demo", "s", "1") 
o.Timeout = 120000 // you can set originate timeout in ms, default 30s
o.Async = true // will execute callback immediately (will not wait for answer)
err := g.Originate(o, nil) // no callback
```**

## Register handler for specific event ##

You can register handler for specific event, example: Hangup, OriginateResponse, MeetmeMute etc.
```
hcb := func(m gami.Message) {
 fmt.Printf("Channel %s was hangupped, cause: %s", m["Channel"], m["Cause"])
}
// will execute callback for each hangup event
err := g.RegisterHandler("Hangup", &hcb) // will return error if callback for such event already registered
...
g.UnregisterHandler("Hangup") // remove handler
```

## Multimessage reponse ##

Some AMI applications may produce several events, example: CoreShowChannels, ConfbridgeList etc. To handle it you can use 'holding' callbacks. Such callbacks will not be deleted after execution, you should handle it by own
```
confch := make(chan gami.Message, 10) // will receive all events for command to this channel
cb := func(m gami.Message) {
 if m["EventList"] == "Complete" || m["Response"] == "Error" {
  close(confch)
  g.DelCallback(m) // should delete callback manually on consuming finish
 } else {
  confch <- m
 }
}

m := gami.Message{"Action": "ConfbridgeList", "Conference": "MyConf"}

g.HoldCallbackAction(m, &cb) // it example, gami already has command GetConfbridgeList

for resp := range <- confch { // iterate on response messages
 do(resp)
}
```