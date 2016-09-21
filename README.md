
YourKit from the Clojure REPL:

I don't always want YourKit profiling enabled, so I have a `:perf` profile that
boots with YourKit. My `project.clj` looks like this. (Note this assumes OSX. You
could adjust the `yjp-file` function according to your system).

```clj
(defn find-file [d f]
  (.trim (:out (clojure.java.shell/sh
                "bash" "-c" (format "find %s -name %s" d f)))))

(defn yjp-file [f]
  (find-file "/Applications/YourKit*" f))

(defn libyjp-jar-path []
  (yjp-file "yjp-controller-api-redist.jar"))

(defn libyjp-agent-path []
  (yjp-file "libyjpagent.jnilib"))

(defproject foo "0.0.0"
   ;; ...
   :profiles {;; ...
              :perf {:resource-paths [~(libyjp-jar-path)]
                     :jvm-opts [~(str "-agentpath:" (libyjp-agent-path))]}})
```

Now you can use this handy macro for profiling:

```clj
(defn yk-cpu-profile-fn [mode f]
  (let [yk (com.yourkit.api.Controller.)]
    (.stopCPUProfiling yk)
    (.clearCPUData yk)
    (case mode
      :sample (.startCPUSampling yk nil)
      :trace (.startCPUTracing yk nil))
    (time (f))
    (.stopCPUProfiling yk)
    nil))

(defmacro with-cpu-profiling [mode & body]
  `(yk-cpu-profile-fn ~mode (fn [] ~@body)))
```

Example:

```
(with-cpu-profiling :trace
    (reduce + (range 1e5)))
```

Assuming YourKit.app is running, you can now click the "Open Snapshot" button to
view the profiling result.

I find this really useful for doing profiling from the REPL.  Hope you enjoy!
