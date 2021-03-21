### 模式

Aeron 以三种模式在 UDP 上运行。

- 第一种是单播（或点对点）模式。
- 第二个是组播模式。
- 第三个是“多目标广播”模式。

这些模式与数据的单向流有关。

Aeron functions over UDP in one of three modes.

- The first is Unicast (or point-to-point) mode.
- The second is Multicast mode.
- The third is Multi-Destination-Cast mode.

These modes pertain to the uni-directional flow of data. It is quite possible that a combined bi-directional connection could be setup that is mixed mode. Unicast in one direction and Multicast in another.

Aeron over UDP may pack more than 1 Frame into a single datagram as it sees fit and the Frame will fit.

###
