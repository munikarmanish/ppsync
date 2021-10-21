priority-based pseudo-sync (ppsync)
===================================

Compared to the vanilla linux kernel 5.4, the changes are as follows:

<details>
<summary>net/core/ppsync.c</summary>

- Defined and exported some global variables and helper functions
    - `PPSYNC_SPLIT`: switch for first-stage splitting
    - `PPSYNC_PORTS`: list high-priority ports
    - `N_PPSYNC_PORTS`: number of items in `PPSYNC_PORTS`
    - `skb_is_high_priority`: checks if packet is high-priority in ingress (rx)
      path
    - `skb_is_high_priority_tx`: checks if packet is high-priority in
      egress (tx) path
</details>


<details>
<summary>drivers/net/ethernet/mellanox/mlx5/core/en_rx.c</summary>

- `mlx5e_handle_rx_cqe`
- `mlx5e_handle_rx_cqe_rep`
- `mlx5e_handle_rx_cqe_mpwrq`
- `mlx5i_handle_rx_cqe`
- `mlx5e_ipsec_handle_rx_cqe`
    - Identify the priority of a packet
    - Handle packets based on priority and whether 1st stage split is enabled
</details>

<details>
<summary>net/ipv4/ip_output.c</summary>

- Set the priority of skb before calling `ip_local_out`
</details>


<details>
<summary>fs/proc/stat.c</summary>

- Add two proc files to enable runtime configuration:
    - /proc/split: toggle first-stage splitting
    - /proc/port: set ports to be regarded as high-priority
</details>


<details>
<summary>include/linux/netdevice.h</summary>

- `struct softnet_data`
    - Added separate packet queues for high-priority packets
</details>


<details>
<summary>include/linux/skbuff.h</summary>

- `struct sk_buff`
    - Added a field to indicate high priority
</details>


<details>
<summary>net/core/dev.c</summary>

- `enqueue_to_backlog`
    - Enqueue high-priority packets to high-priority queue and add the NAPI
      device to the _head_ of the poll_list
- `net_rx_action`
    - Process NAPI devices from poll_list by dequeueing _one at a time_,
      considering the possibility that same device may be in the head of the
      poll_list in the next iteration
- `napi_poll`
    - When adding NAPI device back to poll_list, add to tail _only if_ it is
      not currently already in poll_list
- `process_backlog`
    - If the high-priority packet queue is not empty, process it first
    - Process low-priority packeets _only if_ the high-priority packet queue
      is empty.
</details>