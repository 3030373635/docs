> 大致流程：前端通过websocket发送命令到后端，后端使用paramiko执行命令，通过websocket返回结果

## xtrem

> xterm是一个使用TypeScript编写的前端终端组件

```react
import { Terminal } from "xterm";
import "xterm/css/xterm.css";
import { useCallback, useEffect, useRef } from "react";
import { AttachAddon } from "xterm-addon-attach";

const socketURL = "ws://192.168.137.43:8686/Websocket/Tunnel/BRIGE_SHELL";
function WebTerminal() {
  

  useEffect(() => {
    var term = new Terminal({
      fontFamily: 'Menlo, Monaco, "Courier New", monospace',
      fontWeight: 400,
      fontSize: 14,
      rows: 200,
      cursorStyle: "underline"
    });
    //@ts-ignore
    term.open(document.getElementById("terminal"));
    term.focus();
    async function asyncInitSysEnv() {
        const ws = new WebSocket(socketURL),
        attachAddon = new AttachAddon(ws);
      term.loadAddon(attachAddon);
    }
    asyncInitSysEnv();
    return () => {
      //组件卸载，清除 Terminal 实例
      term.dispose();
    };
  }, []);
  return <div id="terminal"></div>;
}

export default WebTerminal;