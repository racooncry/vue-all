<template>
  <div id="xterm" class="xterm" />
</template>
<script>
import "xterm/css/xterm.css";
import { Terminal } from "xterm";
import { FitAddon } from "xterm-addon-fit";
import { AttachAddon } from "xterm-addon-attach";

export default {
  name: "Xterm",
  props: {
    socketURI: {
      type: String,
      default: "ws://127.0.0.1:8080/webssh"
    }
  },
  mounted() {
    this.initSocket();
  },
  beforeDestroy() {
    this.socket.close();
    this.term.dispose();
  },
  methods: {
    initTerm() {
      this.socket.send(
        JSON.stringify({
          operate: "connect",
          host: "127.0.0.1", //IP
          port: "22", //端口号
          username: "root", //用户名
          password: "123456" //密码*/
        })
      );
      const term = new Terminal({
        cols: 97,
        rows: 37,
        cursorBlink: true, // 光标闪烁
        cursorStyle: "block", // 光标样式  null | 'block' | 'underline' | 'bar'
        scrollback: 800, //回滚
        tabStopWidth: 8, //制表宽度
        screenKeys: true
      });
      const attachAddon = new AttachAddon(this.socket);
      const fitAddon = new FitAddon();
      term.loadAddon(attachAddon);
      term.loadAddon(fitAddon);
      term.open(document.getElementById("xterm"));
      fitAddon.fit();
      term.focus();
      this.term = term;
    },
    initSocket() {
      this.socket = new WebSocket(this.socketURI);
      this.socketOnClose();
      this.socketOnOpen();
      this.socketOnError();
    },
    socketOnOpen() {
      this.socket.onopen = () => {
        // 链接成功后
        this.initTerm();
      };
    },
    socketOnClose() {
      this.socket.onclose = () => {
        // console.log('close socket')
      };
    },
    socketOnError() {
      this.socket.onerror = () => {
        // console.log('socket 链接失败')
      };
    }
  }
};
</script>
