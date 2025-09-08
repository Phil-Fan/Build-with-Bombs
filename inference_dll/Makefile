# Makefile for building inference library and test executable

# Directories
BUILD_LIB_DIR = build_lib
BUILD_TEST_DIR = build_test
CURRENT_DIR = $(shell pwd)

.PHONY: all clean lib test

all: lib test

lib:
	@echo "Building shared library (inference.so)..."
	mkdir -p $(BUILD_LIB_DIR)
	cd $(BUILD_LIB_DIR) && cmake .. && cmake --build . --config Release
	@echo "Moving library to inference_dll directory..."
	@cp $(BUILD_LIB_DIR)/libinference.so $(CURRENT_DIR)/
	@echo "Shared library build complete."

test:
	@echo "Building standalone test executable..."
	mkdir -p $(BUILD_TEST_DIR)
	cd $(BUILD_TEST_DIR) && cmake .. -DSTANDALONE_TEST=ON && cmake --build . --config StandaloneTest
	@echo "Moving executable to inference_dll directory..."
	@cp $(BUILD_TEST_DIR)/inference $(CURRENT_DIR)/
	@echo "Test executable build complete."

ONNX_URL = https://github.com/timothy-barnes-2357/Build-with-Bombs/releases/download/v0.2.1/ddim_single_update.onnx
ONNX_FILE = ddim_single_update.onnx
TENSORRT_PATH=/usr/local/tensorrt-10.5
CUDA_PATH=/usr/local/cuda-12.6

$(ONNX_FILE):
	@echo "Downloading ONNX model..."
	@if [ ! -f $(ONNX_FILE) ]; then \
		wget -q $(ONNX_URL) -O $(ONNX_FILE); \
		echo "Downloaded $(ONNX_FILE)"; \
	else \
		echo "$(ONNX_FILE) already exists"; \
	fi

run: test $(ONNX_FILE)
	@echo "Running inference test..."
	@LD_LIBRARY_PATH=$(TENSORRT_PATH)/lib:$(CUDA_PATH)/lib64:$$LD_LIBRARY_PATH ./inference
	@echo "Test complete.Your Environment is ready."

clean:
	@echo "Cleaning build directories..."
	rm -rf $(BUILD_LIB_DIR)
	rm -rf $(BUILD_TEST_DIR)
	rm -f $(CURRENT_DIR)/libinference.so
	rm -f $(CURRENT_DIR)/inference
	rm -rf buildwithbombs.log
	@echo "Clean complete."
