# === Accel-Sim Usage Steps ===

# 1. Install Accel-Sim following the official guide:
#    https://github.com/accel-sim/accel-sim-framework

# 2. Navigate to the Accel-Sim directory
cd /path/to/accel-sim/

# 3. Set up the simulation environment
source ./gpu-simulator/setup_environment.sh

# 4. Prepare and compile your CUDA application
#    Assume the compiled executable is at: ./path/to/executable

# 5. Run the executable with the tracer to collect execution traces
LD_PRELOAD=./tracer_tool/tracer_tool.so ./path/to/executable

# 6. Post-process the collected traces
./tracer_tool/traces-processing/post-traces-processing ./traces/kernelslist

# 7. Launch the Accel-Sim simulator
#    Replace 'xxxxx' with the name of your selected configuration
./gpu-simulator/bin/release/accel-sim.out \
  -trace ./util/tracer_nvbit/traces/kernelslist.g \
  -config ./gpu-simulator/configs/tested-cfgs/xxxxx/gpgpusim.config \
  -config ./gpu-simulator/configs/tested-cfgs/xxxxx/trace.config \
  > xxx.log 2>&1

# The simulation output will be saved in xxx.log



## Notes on A100 Configuration and Accuracy

To ensure that GEMM operations on tensor cores in Accel-Sim more closely resemble real hardware behavior, we fine-tuned several simulation parameters. These adjustments were validated using selected microbenchmarks from [YH’s Samples](https://github.com/Yinghan-Li/YHs_Sample).

### Adjustments

1. **Tensor Core Throughput**  
   The NVIDIA A100 includes four 8-4-8 tensor cores per cycle. For HMMA instructions with a 16-8-16 data format, the initiation interval (II) is 8 cycles. This behavior is reflected in our customized `trace.config` file to more accurately emulate real execution latency.

2. **L2 Cache Behavior**  
   The A100 features a 40MB L2 cache with larger-than-default bank sizes. Since Accel-Sim’s default IPOLY model does not scale to this size, we adopted a hash-based approximation as suggested by the framework authors. While this method does not represent actual hardware architecture, it provides a reasonable estimate of expected performance.  
   For more details, see this [GitHub issue](https://github.com/accel-sim/accel-sim-framework/issues/126).

3. **Memory Configuration Constraints**  
   Accel-Sim currently does not support non-power-of-two memory sizes. For example, shared memory (smem) and L1 cache must be set to 256 KiB instead of the real-world 192 KiB. This is a known limitation. To avoid related issues, we recommend not allocating memory sizes that depend on the exact hardware capacity.
