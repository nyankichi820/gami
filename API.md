
```
=API=
    
==Types==

* Aid - AMI action id generator

    ** func NewAid() *Aid
        Aid factory
    ** func (a *Aid) Generate() string
        generate action id
    
* Asterisk - main working entity

    ** func NewAsterisk(conn *net.Conn, f *func(error)) *Asterisk
        Asterisk factory
    ** func (a *Asterisk) Bridge(chan1, chan2 string, tone bool, f *func(Message)) error
        bridge two channels already in the PBX
    ** func (a *Asterisk) Command(cmd string, f *func(Message)) error
        execute Asterisk CLI Command
    ** func (a *Asterisk) ConfbridgeKick(conf, chann string, f *func(Message)) error
        kick a Confbridge user
    ** func (a *Asterisk) ConfbridgeList(conference string, f *func(Message)) error
        list participants in a conference (generates multimessage response)
    ** func (a *Asterisk) ConfbridgeStartRecord(conf, file string, f *func(Message)) error
        start conference record
    ** func (a *Asterisk) ConfbridgeStopRecord(conf string, f *func(Message)) error
        stop conference record
    ** func (a *Asterisk) ConfbridgeToggleMute(conf, chann string, mute bool, f *func(Message)) error
        mute/unmute a Confbridge user
    ** func (a *Asterisk) DefaultHandler(f *func(Message))
        set default handler for all Asterisk messages
    ** func (a *Asterisk) DelCallback(m Message)
        delete action callback (used by self-delete callbacks)
    ** func (a *Asterisk) GetConfbridgeList(conference string) ([]Message, error)
        returns conference participants, will blocks execution
    ** func (a *Asterisk) GetMeetmeList(conference string) ([]Message, error)
        returns MeetMe conference participants, blocks execution
    ** func (a Asterisk) Hangup(channel string, f *func(Message)) error
        hangup Asterisk channel
    ** func (a *Asterisk) HoldCallbackAction(m Message, f *func(m Message)) error
        send action with callback which deletes itself (used for multi-message responses) IMPORTANT: callback function must delete itself by own
    ** func (a *Asterisk) Login(login string, password string) error
        logins to AMI and starts read dispatcher
    ** func (a Asterisk) Logoff() error
        logoff from AMI
    ** func (a *Asterisk) MeetmeList(conference string, f *func(Message)) error
        list participants in a MeetMe conference (generates multimessage response) if conference empty string will return for all conferences
    ** func (a *Asterisk) ModuleLoad(module, tload string, f *func(Message)) error
        loads, unloads or reloads an Asterisk module in a running system
    ** func (a *Asterisk) Originate(o *Originate, vars map[string]string, f *func(Message)) error
        make a call
    ** func (a *Asterisk) Reaload(module string, f *func(Message)) error
        reload Asterisk module
    ** func (a Asterisk) Redirect(channel string, context string, exten string, priority string, f *func(Message)) error
        redirect Asterisk channel
    ** func (a *Asterisk) RegisterHandler(event string, f *func(m Message)) error
        register callback for Asterisk event (one handler per event) return err if handler already exists
    ** func (a *Asterisk) SendAction(m Message, f *func(m Message)) error
        universal action send
    ** func (a *Asterisk) UnregisterHandler(event string)
        deregister callback for event
    ** func (a *Asterisk) UserEvent(name string, headers map[string]string, f *func(Message)) error
        send an arbitrary event

* Message map[string]string
    basic gami message

* Originate struct {
    Channel  string // channel to which originate
    Context  string // context to move after originate success
    Exten    string // exten to move after originate success
    Priority string // priority to move after originate success
    Timeout  int    // originate timeout ms
    CallerID string // caller identification string
    Account  string // used for CDR

    Application string // application to execute after successful originate
    Data        string // data passed to application

    Async bool // asynchronous call
}
    Originate, struct used in Originate command if pointed Context and Application, Context has higher priority

** func NewOriginate(channel, context, exten, priority string) *Originate
    Originate entity factory function (to context)
** func NewOriginateApp(channel, app, data string) *Originate
    Originate entity factory function (to application)
```