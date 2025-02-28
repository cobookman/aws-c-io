include(AwsTestHarness)
enable_testing()

# See PKCS11.md for instructions on running these tests
option(ENABLE_PKCS11_TESTS "Build and run PKCS#11 tests" OFF)

file(GLOB TEST_SRC "*.c")
file(GLOB TEST_HDRS "*.h")
file(GLOB TESTS ${TEST_HDRS} ${TEST_SRC})

# Each pipe test runs in 2 different configurations
macro(add_pipe_test_case name)
    add_test_case("${name}")
    add_test_case("${name}_2loops")
endmacro()

add_test_case(io_load_error_strings_test)

add_pipe_test_case(pipe_open_close)
add_pipe_test_case(pipe_read_write)
add_pipe_test_case(pipe_read_write_large_buffer)
add_pipe_test_case(pipe_readable_event_sent_after_write)
add_pipe_test_case(pipe_readable_event_sent_once)
add_pipe_test_case(pipe_readable_event_sent_on_subscribe_if_data_present)
add_pipe_test_case(pipe_readable_event_sent_on_resubscribe_if_data_present)
add_pipe_test_case(pipe_readable_event_sent_again_after_all_data_read)
add_pipe_test_case(pipe_error_event_sent_after_write_end_closed)
add_pipe_test_case(pipe_error_event_sent_on_subscribe_if_write_end_already_closed)
add_pipe_test_case(pipe_writes_are_fifo)
add_pipe_test_case(pipe_clean_up_cancels_pending_writes)

add_test_case(event_loop_xthread_scheduled_tasks_execute)
add_test_case(event_loop_canceled_tasks_run_in_el_thread)
if (USE_IO_COMPLETION_PORTS)
    add_test_case(event_loop_completion_events)
else ()
    add_test_case(event_loop_subscribe_unsubscribe)
    add_test_case(event_loop_writable_event_on_subscribe)
    add_test_case(event_loop_no_readable_event_before_write)
    add_test_case(event_loop_readable_event_after_write)
    add_test_case(event_loop_readable_event_on_subscribe_if_data_present)
    add_test_case(event_loop_readable_event_on_2nd_time_readable)
    add_test_case(event_loop_no_events_after_unsubscribe)
endif ()

add_test_case(event_loop_stop_then_restart)
add_test_case(event_loop_multiple_stops)
add_test_case(event_loop_group_setup_and_shutdown)
add_test_case(event_loop_group_setup_and_shutdown_async)
add_test_case(numa_aware_event_loop_group_setup_and_shutdown)

add_test_case(io_testing_channel)

add_test_case(local_socket_communication)
add_net_test_case(tcp_socket_communication)
add_net_test_case(udp_socket_communication)
add_test_case(udp_bind_connect_communication)
add_net_test_case(connect_timeout)
add_net_test_case(connect_timeout_cancelation)
if (USE_VSOCK)
	add_test_case(vsock_loopback_socket_communication)
endif ()

add_test_case(outgoing_local_sock_errors)
add_test_case(outgoing_tcp_sock_error)
add_test_case(incoming_tcp_sock_errors)
add_test_case(incoming_duplicate_tcp_bind_errors)
add_test_case(incoming_udp_sock_errors)
add_test_case(wrong_thread_read_write_fails)
add_net_test_case(cleanup_before_connect_or_timeout_doesnt_explode)
add_test_case(cleanup_in_accept_doesnt_explode)
add_test_case(cleanup_in_write_cb_doesnt_explode)
add_test_case(sock_write_cb_is_async)

if (WIN32)
    add_test_case(local_socket_pipe_connected_race)
endif()

add_test_case(channel_setup)
add_test_case(channel_single_slot_cleans_up)
add_test_case(channel_slots_clean_up)
add_test_case(channel_refcount_delays_clean_up)
add_test_case(channel_tasks_run)
add_test_case(channel_rejects_post_shutdown_tasks)
add_test_case(channel_cancels_pending_tasks)
add_test_case(channel_duplicate_shutdown)
add_net_test_case(channel_connect_some_hosts_timeout)

add_net_test_case(test_default_with_ipv6_lookup)
add_test_case(test_resolver_ipv6_address_lookup)
add_net_test_case(test_default_with_multiple_lookups)
add_test_case(test_resolver_ipv4_address_lookup)
add_net_test_case(test_default_with_ipv4_only_lookup)
add_test_case(test_resolver_ttls)
add_test_case(test_resolver_connect_failure_recording)
add_test_case(test_resolver_ttl_refreshes_on_resolve)

add_net_test_case(test_resolver_listener_create_destroy)
add_net_test_case(test_resolver_add_listener_before_host)
add_net_test_case(test_resolver_add_listener_after_host)
add_net_test_case(test_resolver_add_multiple_listeners_fn)
add_net_test_case(test_resolver_listener_host_re_add_fn)
add_net_test_case(test_resolver_listener_multiple_results)
add_net_test_case(test_resolver_listener_address_expired_fn)
add_net_test_case(test_resolver_pinned_host_entry)
add_net_test_case(test_resolver_unpinned_host_entry)
add_net_test_case(test_resolver_address_promote_demote_listener_callbacks)

add_test_case(test_pem_single_cert_parse)
add_test_case(test_pem_private_key_parse)
add_test_case(test_pem_cert_chain_parse)
add_test_case(test_pem_cert_parse_from_file)
add_test_case(test_pem_private_key_parse_from_file)
add_test_case(test_pem_cert_chain_comments_and_whitespace)
add_test_case(test_pem_invalid_parse)
add_test_case(test_pem_valid_data_invalid_parse)
add_test_case(test_pem_invalid_in_chain_parse)

add_test_case(pem_sanitize_comments_around_pem_object_removed)
add_test_case(pem_sanitize_empty_file_rejected)
add_test_case(pem_sanitize_wrong_format_rejected)

add_test_case(socket_handler_echo_and_backpressure)
add_test_case(socket_handler_close)
add_test_case(socket_pinned_event_loop)

if (NOT BYO_CRYPTO)
    # Badssl-based tests (https://badssl.com/dashboard/) - use remote endpoints for now, later transition to
    # internal hosting using the bad ssl container/server setup and dns redirects per
    # https://github.com/chromium/badssl.com
    #
    # We don't use the interception suite since that's a host configuration issue
    # We also don't check the domain security policy suite:
    #  1. s2n does not support revocation checks and we do not currently enable any revocation checks
    #    in windows or osx, although we may add configurable support to those platforms(off-by-default) at a
    #    later date
    #  2. s2n does not support public key pinning and, given its deprecated and http-centric position, there are no
    #    plans to add support nor investigate osx/windows support as well.

    # Badssl - Certificate Validation endpoint suite
    # For each failure case, we also include a positive test that verifies success when peer verification is disabled
    add_net_test_case(tls_client_channel_negotiation_error_expired)
    add_net_test_case(tls_client_channel_negotiation_error_wrong_host)
    add_net_test_case(tls_client_channel_negotiation_error_wrong_host_with_ca_override)
    add_net_test_case(tls_client_channel_negotiation_error_self_signed)
    add_net_test_case(tls_client_channel_negotiation_error_untrusted_root)
    add_net_test_case(tls_client_channel_negotiation_error_untrusted_root_due_to_ca_override)
    add_net_test_case(tls_client_channel_negotiation_no_verify_expired)
    add_net_test_case(tls_client_channel_negotiation_no_verify_wrong_host)
    add_net_test_case(tls_client_channel_negotiation_no_verify_self_signed)
    add_net_test_case(tls_client_channel_negotiation_no_verify_untrusted_root)

    # Badssl - Broken Crypto endpoint suite
    # We don't include dh1024 as it succeeds on the windows baseline configuration and there does not seem
    # to be a way to disable it
    add_net_test_case(tls_client_channel_negotiation_error_broken_crypto_rc4)
    add_net_test_case(tls_client_channel_negotiation_error_broken_crypto_rc4_md5)
    add_net_test_case(tls_client_channel_negotiation_error_broken_crypto_dh480)
    add_net_test_case(tls_client_channel_negotiation_error_broken_crypto_dh512)
    add_net_test_case(tls_client_channel_negotiation_error_broken_crypto_null)

    # Badssl - Legacy crypto suite, includes both negative and positive tests, with override checks where appropriate
    # Our current baseline/default is platform-specific, whereas badssl expects a baseline of 1.2
    #   Linux - tls1.1
    #   Windows - system default (1.0 is the only thing we could reasonable fixate to given win7 support)
    #   Mac - system default
    # We skip the cbc and 3des checks, as a positive connection result there does not yet represent a security risk
    # We don't include dh2048 as it succeeds on the windows baseline configuration and there does not seem
    # to be a way to disable it
    add_net_test_case(tls_client_channel_negotiation_error_legacy_crypto_tls10)
    add_net_test_case(tls_client_channel_negotiation_override_legacy_crypto_tls10)
    add_net_test_case(tls_client_channel_negotiation_error_override_legacy_crypto_tls11)
    add_net_test_case(tls_client_channel_negotiation_success_legacy_crypto_tls11)

    # Badssl - Secure uncommon suite
    # We skip 10000san for now as its unclear the point or relevance especially with respect to the OS-based
    # TLS implementations
    # We skip 1000san because the public badssl certificate is expired and we haven't migrated to internal hosting yet
    # We also defer the incomplete chain test for now until we can do some further study on how to get it to
    # properly fail on windows and osx.
    add_net_test_case(tls_client_channel_negotiation_success_sha384)
    add_net_test_case(tls_client_channel_negotiation_success_sha512)
    add_net_test_case(tls_client_channel_negotiation_success_rsa8192)
    add_net_test_case(tls_client_channel_negotiation_error_no_subject)
    add_net_test_case(tls_client_channel_negotiation_success_no_verify_no_subject)
    add_net_test_case(tls_client_channel_negotiation_error_no_common_name)
    add_net_test_case(tls_client_channel_negotiation_success_no_verify_no_common_name)
    add_net_test_case(tls_client_channel_negotiation_success_no_verify_incomplete_chain)

    # Badssl - Secure common suite, all of these should succeed
    add_net_test_case(tls_client_channel_negotiation_success_tls12)
    add_net_test_case(tls_client_channel_negotiation_success_sha256)
    add_net_test_case(tls_client_channel_negotiation_success_rsa2048)
    add_net_test_case(tls_client_channel_negotiation_success_ecc256)
    add_net_test_case(tls_client_channel_negotiation_success_ecc384)
    add_net_test_case(tls_client_channel_negotiation_success_extended_validation)
    add_net_test_case(tls_client_channel_negotiation_success_mozilla_modern)

    # Misc non-badssl tls tests
    add_net_test_case(test_concurrent_cert_import)
    add_test_case(tls_channel_echo_and_backpressure_test)
    add_net_test_case(tls_client_channel_negotiation_error_socket_closed)
    add_net_test_case(tls_client_channel_negotiation_success)
    add_net_test_case(tls_server_multiple_connections)
    add_net_test_case(tls_server_hangup_during_negotiation)
    add_net_test_case(tls_client_channel_no_verify)
    add_net_test_case(test_tls_negotiation_timeout)
    add_net_test_case(alpn_successfully_negotiates)
    add_net_test_case(alpn_no_protocol_message)
    add_net_test_case(test_ecc_cert_import)

    add_test_case(alpn_error_creating_handler)
    add_test_case(tls_destroy_null_context)
    add_test_case(tls_channel_statistics_test)
else()
    add_test_case(byo_tls_handler_test)
endif()


add_test_case(uri_full_parse)
add_test_case(uri_no_scheme_parse)
add_test_case(uri_no_port_parse)
add_test_case(uri_no_path_parse)
add_test_case(uri_no_query_parse)
add_test_case(uri_minimal_parse)
add_test_case(uri_root_only_parse)
add_test_case(uri_path_and_query_only_parse)
add_test_case(uri_userinfo_no_password_parse)
add_test_case(uri_empty_user_parse)
add_test_case(uri_query_params)
add_test_case(uri_invalid_scheme_parse)
add_test_case(uri_invalid_port_parse)
add_test_case(uri_port_too_large_parse)
add_test_case(uri_builder)
add_test_case(uri_builder_from_string)
add_test_case(test_uri_encode_path_rfc3986)
add_test_case(test_uri_encode_query)
add_test_case(test_uri_decode)

add_test_case(test_input_stream_memory_simple)
add_test_case(test_input_stream_memory_iterate)
add_test_case(test_input_stream_memory_seek_beginning)
add_test_case(test_input_stream_memory_seek_end)
add_test_case(test_input_stream_memory_seek_multiple_times)
add_test_case(test_input_stream_memory_seek_past_end)
add_test_case(test_input_stream_memory_seek_before_start)
add_test_case(test_input_stream_file_simple)
add_test_case(test_input_stream_file_iterate)
add_test_case(test_input_stream_file_seek_beginning)
add_test_case(test_input_stream_file_seek_end)
add_test_case(test_input_stream_memory_length)
add_test_case(test_input_stream_file_length)
add_test_case(test_input_stream_binary)

add_test_case(open_channel_statistics_test)

add_test_case(shared_library_open_failure)

if (BUILD_SHARED_LIBS)
    add_test_case(shared_library_open_success)
    add_test_case(shared_library_find_function_failure)
    add_test_case(shared_library_find_function_success)
endif()

add_test_case(test_exponential_backoff_retry_too_many_retries_no_jitter)
add_test_case(test_exponential_backoff_retry_too_many_retries_full_jitter)
add_test_case(test_exponential_backoff_retry_too_many_retries_decorrelated_jitter)
add_test_case(test_exponential_backoff_retry_too_many_retries_default_jitter)
add_test_case(test_exponential_backoff_retry_client_errors_do_not_count)
add_test_case(test_exponential_backoff_retry_no_jitter_time_taken)
add_test_case(test_exponential_backoff_retry_invalid_options)

add_test_case(test_standard_retry_strategy_setup_shutdown)
add_test_case(test_standard_retry_strategy_failure_exhausts_bucket)
add_test_case(test_standard_retry_strategy_failure_recovers)

# See PKCS11.md for instructions on running these tests
if (ENABLE_PKCS11_TESTS)
    add_test_case(pkcs11_lib_sanity_check)
    add_test_case(pkcs11_lib_behavior_default)
    add_test_case(pkcs11_lib_behavior_omit_initialize)
    add_test_case(pkcs11_lib_behavior_strict_initialize_finalize)
    add_test_case(pkcs11_find_private_key)
    add_test_case(pkcs11_find_private_key_for_different_types)
    add_test_case(pkcs11_find_multiple_private_key)
    add_test_case(pkcs11_sign_sha1)
    add_test_case(pkcs11_sign_sha224)
    add_test_case(pkcs11_sign_sha256)
    add_test_case(pkcs11_sign_sha384)
    add_test_case(pkcs11_sign_sha512)
    add_test_case(pkcs11_decrypt)
    add_test_case(pkcs11_find_slot)
    add_test_case(pkcs11_find_slot_many_tokens)
    add_test_case(pkcs11_session_tests)
    add_test_case(pkcs11_login_tests)

    # TLS with PKCS#11 not currently supported on every platform
    if (USE_S2N)
        add_test_case(pkcs11_tls_negotiation_succeeds)
    endif()
endif()

set(TEST_BINARY_NAME ${PROJECT_NAME}-tests)
generate_test_driver(${TEST_BINARY_NAME})

#SSL certificates to use for testing.
add_custom_command(TARGET ${TEST_BINARY_NAME} PRE_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_SOURCE_DIR}/resources $<TARGET_FILE_DIR:${TEST_BINARY_NAME}>)
