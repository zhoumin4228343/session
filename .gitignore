package session

import (
	"crypto/rand"
	"encoding/base64"
	"errors"
	"fmt"
	"runtime"
	"strconv"
	"time"
)

type Session string

const (
	base64Table = "123QRSTUabcdVWXYZHijKLAWDCABDstEFGuvwxyzGHIJklmnopqr234560178912"
)

var (
	sessions       = make(map[string]map[string]interface{})
	sessionTimeout = time.Second * 120 //default expire session after 1 hour
)

func New() Session {
	sid, err := generateSessionId()
	if err != nil {
		errors.New("session id not create")
	}
	sessions[sid] = make(map[string]interface{})
	t := time.NewTimer(sessionTimeout)
	tc := make(chan int)
	sessions[sid]["tc"] = tc
	go startSession(t, tc, sid)
	return Session(sid)
}

func SessionFromId(sid string) Session {
	return Session(sid)
}

func (s Session) Resettimeout() error {
	sobj, ok := sessions[string(s)]
	if !ok {
		return errors.New("Session does not exist")
	}

	tc := sobj["tc"]
	if re, ok := tc.(chan int); ok {
		re <- 1
	} else {
		return errors.New("reset is error,tc is not chan")
	}
	return nil
}

func (s Session) RemoveSession() {
	delete(sessions, string(s))
}

func (s Session) Exists() bool {
	_, ok := sessions[string(s)]
	return ok
}

func (s Session) Put(name string, value interface{}) error {
	sobj, ok := sessions[string(s)]
	if !ok {
		return errors.New("Session does not exist.")
	}

	sobj[name] = value
	return nil
}

func (s Session) Get(name string) (interface{}, error) {
	sobj, ok := sessions[string(s)]
	if !ok {
		return nil, errors.New("Session does not exist.")
	}

	var v interface{}
	v, ok = sobj[name]
	if !ok {
		return nil, errors.New("No such key.")
	}
	return v, nil
}

func startSession(t *time.Timer, tc chan int, sid string) {
h:
	for {
		select {
		case <-t.C:
			t.Stop()
			fmt.Printf("users's session | %s | was deleted\n", sid)
			delete(sessions, sid)
			//删除掉的,要直接退出go携程
			fmt.Println("goroutine count:" + strconv.FormatInt(int64(runtime.NumGoroutine()), 10))
			break h
		case <-tc:
			t.Reset(sessionTimeout)
			fmt.Printf("users's session | %s |  timeout is reset\n", sid)
		}
	}
}

func generateSessionId() (string, error) {
	bytes := make([]byte, 32)
	if _, err := rand.Read(bytes); err != nil {
		return "", err
	}

	coder := base64.NewEncoding(base64Table)
	return coder.EncodeToString(bytes), nil
}
