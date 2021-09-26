### 集成 metrics, traces, logs

进行这方面的集成可以有两种思路：

- 基于 Grafana 这个平台，开发给 pprof 的 panel 插件，这样可以直接展示出 profiles。由于 Conprof 也是基于 labels 的系统，所以可以和 Prometheus， Loki 等系统进行联动。

- 基于 Exemplars，前面提到了 Exemplars 包含了标签对，也就是说我们只需要将单独一个类似于 ID 一样的东西来唯一标识一个 profile， 然后通过 Exemplar 来将 metrics 链接到对应的 profile。
