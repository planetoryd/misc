12 21:54:51 nsproxy[272523]: 2023-12-12T13:54:51.988370Z  INFO nsproxy: Waiting for device FD
12 21:54:52 nsproxy[272523]: 2023-12-12T13:54:52.003157Z  INFO nsproxy: Got FD
12 21:54:52 nsproxy[272523]: 2023-12-12T13:54:52.004019Z  INFO nsproxy: IArgs { proxy: ArgProxy { proxy_type: Socks5, addr: 127.0.0.1:9902, credentials: None }, ipv6_enabled: true, dns: Handled, dns_addr: 127.0.0.1, bypass: [] }
12 21:54:52 nsproxy[272523]: 2023-12-12T13:54:52.006571Z  WARN ipstack: parse error UnsupportedTransportProtocol
12 21:54:55 nsproxy[272523]: 2023-12-12T13:54:55.100266Z TRACE tun2socks5: Session count 1
12 21:54:55 nsproxy[272523]: 2023-12-12T13:54:55.100676Z  INFO tun2socks5::dns: Allocate VirtDNS Name example.com.
12 21:54:55 nsproxy[272523]: 2023-12-12T13:54:55.101215Z  INFO tun2socks5::dns: Allocate VirtDNS Name example.com.
12 21:54:55 nsproxy[272523]: 2023-12-12T13:54:55.102506Z TRACE tun2socks5: Session count 2
12 21:54:55 nsproxy[272523]: 2023-12-12T13:54:55.103011Z  INFO tun2socks5: Beginning #1 TCP 100.64.0.2:47438 -> example.com.:80
12 21:54:55 nsproxy[272523]: [ipstack/src/stream/tcp.rs:243] self.tcb.get_state() = SynReceived(
12 21:54:55 nsproxy[272523]:     true,
12 21:54:55 nsproxy[272523]: )
12 21:54:55 nsproxy[272523]: [ipstack/src/stream/tcp.rs:243] self.tcb.get_state() = Established
12 21:54:55 nsproxy[272523]: [ipstack/src/stream/tcp.rs:243] self.tcb.get_state() = Established
12 21:54:55 nsproxy[272523]: [ipstack/src/stream/tcb.rs:157] received_ack_distance > current_ack_distance = false
12 21:54:55 nsproxy[272523]: [ipstack/src/stream/tcb.rs:158] incoming_packet.inner().acknowledgment_number != self.seq = true
12 21:54:55 nsproxy[272523]: [ipstack/src/stream/tcb.rs:159] incoming_packet.inner().acknowledgment_number.saturating_sub(self.seq) == 0 = true
12 21:54:55 nsproxy[272523]: 2023-12-12T13:54:55.999647Z  WARN ipstack::stream::tcb: tcp packet invalid, dist
12 21:54:55 nsproxy[272523]: [ipstack/src/stream/tcp.rs:243] self.tcb.get_state() = Established
12 21:54:56 nsproxy[272523]: [ipstack/src/stream/tcp.rs:243] self.tcb.get_state() = Established
12 21:54:56 nsproxy[272523]: [ipstack/src/stream/tcp.rs:243] self.tcb.get_state() = FinWait2
12 21:54:56 nsproxy[272523]: [ipstack/src/stream/tcb.rs:157] received_ack_distance > current_ack_distance = true
12 21:54:56 nsproxy[272523]: [ipstack/src/stream/tcb.rs:158] incoming_packet.inner().acknowledgment_number != self.seq = true
12 21:54:56 nsproxy[272523]: [ipstack/src/stream/tcb.rs:159] incoming_packet.inner().acknowledgment_number.saturating_sub(self.seq) == 0 = false
12 21:54:56 nsproxy[272523]: 2023-12-12T13:54:56.000828Z  WARN ipstack::stream::tcb: tcp packet invalid, dist
12 21:54:56 nsproxy[272523]: 2023-12-12T13:54:56.312005Z  WARN ipstack: parse error UnsupportedTransportProtocol
12 21:55:01 nsproxy[272523]: 2023-12-12T13:55:01.001655Z  WARN ipstack::stream::tcp: Timeout reached for 198.18.0.0:80
12 21:55:01 nsproxy[272523]: 2023-12-12T13:55:01.001843Z  INFO tun2socks5: Ending #1 TCP 100.64.0.2:47438 -> example.com.:80 with Err(Kind(TimedOut))
12 21:55:01 nsproxy[272523]: 2023-12-12T13:55:01.002159Z TRACE tun2socks5: Session count 1
12 21:55:01 nsproxy[272523]: 2023-12-12T13:55:01.002514Z  WARN ipstack: ttl 0 remove stream NetworkTuple { src: 100.64.0.2:47438, dst: 198.18.0.0:80, tcp: true }
12 21:55:04 nsproxy[272523]: 2023-12-12T13:55:04.418757Z  WARN ipstack: parse error UnsupportedTransportProtocol
12 21:55:21 nsproxy[272523]: 2023-12-12T13:55:21.275141Z  WARN ipstack: parse error UnsupportedTransportProtocol
12 21:55:53 nsproxy[272523]: 2023-12-12T13:55:53.698672Z  WARN ipstack: parse error UnsupportedTransportProtocol
12 21:56:55 nsproxy[272523]: 2023-12-12T13:56:55.141758Z  WARN ipstack: parse error UnsupportedTransportProtocol


```rust
// FIXME
    pub(super) fn check_pkt_type(&self, incoming_packet: &TcpPacket, p: &[u8]) -> PacketStatus {
        let received_ack_distance = self
            .seq
            .wrapping_sub(incoming_packet.inner().acknowledgment_number);

        let current_ack_distance = self.seq.wrapping_sub(self.last_ack);
        if received_ack_distance > current_ack_distance
            || (incoming_packet.inner().acknowledgment_number != self.seq
                && incoming_packet
                    .inner()
                    .acknowledgment_number
                    .saturating_sub(self.seq)
                    == 0)
        {
            dbg!(received_ack_distance > current_ack_distance);
            dbg!(incoming_packet.inner().acknowledgment_number != self.seq);
            dbg!(incoming_packet
                .inner()
                .acknowledgment_number
                .saturating_sub(self.seq)
                == 0);
            warn!("tcp packet invalid, dist");
            PacketStatus::Invalid
        } else if self.last_ack == incoming_packet.inner().acknowledgment_number {
            if !p.is_empty() {
                PacketStatus::NewPacket
            } else if self.send_window == incoming_packet.inner().window_size
                && self.seq != self.last_ack
            {
                PacketStatus::RetransmissionRequest
            } else {
                PacketStatus::WindowUpdate
            }
        } else if self.last_ack < incoming_packet.inner().acknowledgment_number {
            if !p.is_empty() {
                PacketStatus::NewPacket
            } else {
                PacketStatus::Ack
            }
        } else {
            warn!("tcp packet invalid");
            PacketStatus::Invalid
        }
    }
```