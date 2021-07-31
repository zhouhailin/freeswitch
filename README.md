# FreeSWITCH 源代码分析

源代码分析和注释不修改源代码，只添加相关注释说明，源代码行数会受影像

## 编译安装

### Clone源代码

#### 最新代码

    git clone https://github.com/signalwire/freeswitch

#### 指定分支

    git clone -b v1.6 https://github.com/signalwire/freeswitch

### 源代码编译安装

#### 首次

    ./bootstrap.sh && ./configure && make && make install

#### 增量

##### 所有模块

    make && make install

##### 指定模块

    # 指定模块调整
    make mod_sofia-install

    # 如果核⼼的代码调整
    make core-install

## 源码目录

## 源码分析

主要包含启动、模块、呼叫

### 启动

    int main(int argc, char *argv[]) // switch.c : 主程序启动入口
        switch_core_init_and_modload(flags, nc ? SWITCH_FALSE : SWITCH_TRUE, &err) // switch.c/switch_core.c : 初始化并加载模块
            switch_core_init(flags, console, err);
                memset(&runtime, 0, sizeof(runtime)); // runtime 内存申请后初始化相关参数
                sqlite3_initialize()        // SQLite 初始化
                apr_initialize()            // APR 初始化创建内存池上下文
                switch_core_memory_init()   // 申请内存池
                switch_dir_make_recursive(SWITCH_GLOBAL_dirs.*, SWITCH_DEFAULT_DIR_PERMS, runtime.memory_pool); // 创建运行时所需目录
                switch_core_set_globals();
                switch_core_session_init(runtime.memory_pool);
                switch_event_create_plain(&runtime.global_vars, SWITCH_EVENT_CHANNEL_DATA);
                switch_core_hash_init_case(&runtime.*, SWITCH_FALSE);
                load_mime_types();
                SSL_library_init();
                switch_ssl_init_ssl_locks();
                switch_curl_init();
                switch_core_set_variable("hostname", runtime.hostname); // 设置相关全局变量
                ENABLE_ZRTP : switch_core_set_serial();
                switch_console_init(runtime.memory_pool);
                switch_event_init(runtime.memory_pool);
                switch_channel_global_init(runtime.memory_pool);
                switch_xml_init(runtime.memory_pool, err); && apr_terminate();
                switch_nat_init(runtime.memory_pool, switch_test_flag((&runtime), SCF_USE_NAT_MAPPING));
                switch_log_init(runtime.memory_pool, runtime.colorize_console);
                switch_load_core_config("switch.conf");
                switch_core_state_machine_init(runtime.memory_pool);
                switch_core_sqldb_start(runtime.memory_pool, switch_test_flag((&runtime), SCF_USE_SQL);
                switch_core_media_init();
                switch_scheduler_task_thread_start();
                switch_nat_late_init();
                switch_rtp_init(runtime.memory_pool);
                switch_scheduler_add_task(switch_epoch_time_now(NULL), heartbeat_callback, "heartbeat", "core", 0, NULL, SSHF_NONE | SSHF_NO_DEL);
                switch_scheduler_add_task(switch_epoch_time_now(NULL), check_ip_callback, "check_ip", "core", 0, NULL, SSHF_NONE | SSHF_NO_DEL | SSHF_OWN_THREAD);
                switch_uuid_get(&uuid);
                switch_uuid_format(runtime.uuid_str, &uuid);
                switch_core_set_variable("core_uuid", runtime.uuid_str);
            switch_core_set_signal_handlers();
            switch_load_network_lists(SWITCH_FALSE);
            switch_loadable_module_init(SWITCH_TRUE); // switch_loadable_module.c
                memset(&loadable_modules, 0, sizeof(loadable_modules));
                switch_core_new_memory_pool(&loadable_modules.pool);
                switch_core_hash_init(&loadable_modules.module_hash);
                switch_core_hash_init_nocase(&loadable_modules.*);
                switch_core_hash_init(&loadable_modules.file_hash);
                switch_core_hash_init_nocase(&loadable_modules.speech_hash);
                switch_loadable_module_load_module("", "CORE_SOFTTIMER_MODULE", SWITCH_FALSE, &err);
                switch_loadable_module_load_module("", "CORE_PCM_MODULE", SWITCH_FALSE, &err);
                switch_loadable_module_load_module("", "CORE_SPEEX_MODULE", SWITCH_FALSE, &err);
                SWITCH_HAVE_YUV && SWITCH_HAVE_VPX : switch_loadable_module_load_module("", "CORE_VPX_MODULE", SWITCH_FALSE, &err);
                switch_xml_open_cfg("modules.conf", &cfg, NULL);
                    switch_core_hash_find_locked(loadable_modules.module_hash, file, loadable_modules.mutex); // false : Module already loaded
                    switch_loadable_module_load_file(path, file, global, &new_module));
                    switch_loadable_module_process(file, new_module));
                    switch_core_launch_thread(switch_loadable_module_exec, new_module, new_module->pool);
                switch_xml_open_cfg("post_load_modules.conf", &cfg, NULL)
                    switch_core_hash_find_locked(loadable_modules.module_hash, file, loadable_modules.mutex); // false : Module already loaded
                    switch_loadable_module_load_file(path, file, global, &new_module));
                    switch_loadable_module_process(file, new_module));
                    switch_core_launch_thread(switch_loadable_module_exec, new_module, new_module->pool);
                switch_loadable_module_runtime();
                    switch_core_hash_first(loadable_modules.module_hash);
                    switch_core_launch_thread(switch_loadable_module_exec, module, loadable_modules.pool);
            switch_load_network_lists(SWITCH_FALSE);
            switch_load_core_config("post_load_switch.conf");
            switch_core_set_signal_handlers();
            switch_event_create(&event, SWITCH_EVENT_STARTUP); && switch_event_fire(&event); // System Ready
            switch_core_screen_size(&x, NULL);
            switch_core_get_variable_dup("api_on_startup"); && switch_console_execute(cmd, 0, &stream)
        switch_core_runtime_loop(nc); // switch.c/switch_core.c : 运行阶段
            bg : WaitForSingleObject(shutdown_event, INFINITE);   // switch_core.c : 后台运行 - Windows
            bg : while (runtime.running) {switch_yield(1000000);} // switch_core.c : 后台运行 - Linux
            switch_console_loop(); // switch_core.c/switch_console.c : 前台运行时, 控制台循环读取
        switch_core_destroy(); // switch.c/switch_core.c : 停止阶段
            switch_event_create(&event, SWITCH_EVENT_SHUTDOWN); && switch_event_fire(&event); // switch_core.c : 创建关机事件并分发
            switch_core_session_hupall(SWITCH_CAUSE_SYSTEM_SHUTDOWN); // switch_core.c : 关闭所有会话通道
            switch_loadable_module_shutdown(); // switch_core.c : 停止所有模块
            switch_ssl_destroy_ssl_locks();
            switch_core_sqldb_stop()
            switch_scheduler_task_thread_stop(); // 停止所有定时任务
            switch_rtp_shutdown();
            switch_nat_shutdown();
            switch_xml_destroy();
            switch_console_shutdown();
            switch_channel_global_uninit();
            switch_event_shutdown();
            switch_log_shutdown();
            switch_core_session_uninit();
            switch_core_unset_variables();
            switch_core_memory_stop();
            switch_safe_free(SWITCH_GLOBAL_dirs.*);
            switch_event_destroy(&runtime.*);
            switch_core_hash_destroy(&runtime.*);
            switch_core_destroy_memory_pool(&IP_LIST.pool);
            switch_core_media_deinit();
            apr_pool_destroy(runtime.memory_pool);
            sqlite3_shutdown();

### 模块

### 呼叫

