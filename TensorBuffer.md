# What is TensorBuffer?
TensorBuffer is a core data structure in Google's LiteRT (formerly TensorFlow Lite) API that represents input and output tensors for machine learning models.

Simple Explanation
Think of TensorBuffer as a container for multi-dimensional arrays that hold the data going into and coming out of your ML model. It's essentially a wrapper around native memory buffers that:

Stores tensor data (images, audio, numbers, etc.)
Manages memory allocated for that data
Provides type-safe access to read/write the data
Handles hardware acceleration by interfacing with NPU/GPU/CPU memory
In Your Code
kotlin
// INPUT: Writing data TO the model
inputBuffers[0].writeFloat(preprocessedImageData)  // TensorBuffer
model.run(inputBuffers, outputBuffers)
// OUTPUT: Reading data FROM the model  
val outputArray = outputBuffers[0].readFloat()  // TensorBuffer crashes here!
Why It's Important in This Bug
The crash happens specifically when calling TensorBuffer.readFloat() because:

Memory Location: The output tensor lives in shared memory between the CPU and hardware accelerators (NPU/GPU)
SM8850 Issue: Snapdragon 8 Elite has stricter memory protection for these shared regions
Access Violation: LiteRT 2.1.1 tries to read the buffer but doesn't have proper memory permissions on SM8850
SIGSEGV Result: The OS blocks the access → crash with SEGV_ACCERR (access error)

### Technical Details
 kotlin

// TensorBuffer interface (simplified)
```
class TensorBuffer {
    // Read methods (what we're using)
    fun readFloat(): FloatArray       // ❌ Crashes on SM8850 NPU/GPU
    fun readInt(): IntArray
    fun readByte(): ByteArray
    
    // Write methods
    fun writeFloat(data: FloatArray)  // ✅ Works fine
    fun writeInt(data: IntArray)
    
    // Internal
    private val nativeHandle: Long    // Pointer to native memory
    val shape: IntArray               // Tensor dimensions [513, 513, 21]
    val dataType: DataType            // FLOAT32, INT8, etc.
}

```

Why Both NPU and GPU Crash at Same Address
The same TensorBuffer implementation is used for all accelerators. The crash location (0xb400007397d71000) is in the shared LiteRT runtime code, not accelerator-specific drivers. This proves it's a LiteRT API bug, not a hardware problem.

The Missing API
Ideally, LiteRT should provide:

kotlin
```
// Current (crashes on SM8850)
val data = outputBuffers[0].readFloat()
// Safer alternative (doesn't exist)
val buffer = ByteBuffer.allocateDirect(size)
outputBuffers[0].read(buffer)  // ❌ This method doesn't exist!
```

This would let developers manually allocate memory with proper permissions, but the API doesn't support it yet.

In summary: TensorBuffer is just a smart array wrapper for ML model data, but on SM8850 it has a memory permission bug when reading from hardware-accelerated outputs.
