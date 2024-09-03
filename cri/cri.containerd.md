---
id: cri.containerd
tags:
- containerd
- k8s
- kubernetes
title: "Containerd \u914D\u7F6E\u6587\u4EF6"

---
# Containerd 配置文件
Containerd 的参考配置文件：

```toml
disabled_plugins = []
imports = []
oom_score = -999
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd/"
version = 2

[cgroup]
  path = ""

[debug]
  address = ""
  format = ""
  uid = 0
  gid = 0
  level = ""

[grpc]
  address = "/run/containerd/containerd.sock"
  uid = 0
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""

[metrics]
  address = ""
  grpc_histogram = false

[plugins]

  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_deplay = "0s"
    startup_delay = "100ms"

  [plugins."io.containerd.grpc.v1.cri"]
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    enable_selinux = false
    ignore_image_defined_volumes = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "pause:3.5"
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""

    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d/"
      conf_template = ""
      max_conf_num = 1
  
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "nvidia"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      no_pivot = false
      snapshotter = "overlayfs"
  
      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        base_runtime_spec = ""
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_root = ""
        runtime_type = ""
        
        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]
    
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          base_runtime_spec = ""
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
  
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v1"
          
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
            SystemdCgroup = true
          
        [plugins."io.containerd.grpc.v1.cri".containerd.untrust_workdload_runtime]
          base_runtime_spec = ""
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = ""
          
          [plugins."io.containerd.grpc.v1.cri".containerd.untrust_workdload_runtime.options]
    
    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"
  
    [plugins."io.containerd.grpc.v1.cri".registry]
    
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

        [plugins."io.containerd.grpc.v1.cri".registry."docker.io"]
           endpoint = ["https://ustc-edu-cn.mirrors.aliyuncs.com"]
      
        [plugins."io.containerd.grpc.v1.cri".registry."harbor.mydomain.com"]
          endpoint = ["https://harbor.mydomain.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.configs]
          
          [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.mydomain.com".tls]
            insecure_skip_verify = true
          [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.mydomain.com".auth]
            username = "******"
            password = "******"
    
    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""

  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"

  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"
  
  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"
  
  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false
  
  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "container-shim"
    shim_debug = false

  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
  
  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]
  
  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""
  
  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""
  
  [plugins."io.containerd.snapshotter.v1.devmapper"]
    root_path = ""
    base_image_size = ""
    pool_name = ""
    async_remove = true
  
  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""
  
  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    root_path = ""
  
  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""

[proxy_plugins]

[stream_processors]

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    return = "application/vnd.oci.image.layer.v1.tar"

  [stream_processors."application/vnd.oci.image.layer.v1.tar+gzip"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    return = "application/vnd.oci.image.layer.v1.tar+gzip"

[timeouts]
  "io.containerd.timemout.shim.cleanup" = "5s"
  "io.containerd.timemout.shim.load" = "5s"
  "io.containerd.timemout.shim.shutdown" = "3s"
  "io.containerd.timemout.shim.state" = "2s"

[ttrpc]
  address = ""
  uid = 0
  git = 0
  
```

