# Stage 1: Build test-launch from gst-rtsp-server
# Use trixie-slim as the base image for a lightweight footprint
FROM debian:trixie-slim AS builder

# Build deps
# Install build tools and GStreamer packages needed to compile test-launch
RUN apt-get update && apt-get install -y \
    build-essential git meson ninja-build pkg-config \
    libglib2.0-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
    libgstrtspserver-1.0-dev \
    # Install runtime plugins needed for video pipeline
    gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-libav \
    && rm -rf /var/lib/apt/lists/*


# Clone the gst-rtsp-server source and compile the examples
WORKDIR /usr/src/gst-rtsp-server
RUN git clone https://gitlab.freedesktop.org/gstreamer/gst-rtsp-server.git . \
    && meson setup build --buildtype=release \
        -Dexamples=enabled \
        -Dtests=disabled \
        -Dintrospection=disabled \
    && meson compile -C build examples/test-launch


# ------------------------------------------------------------------
# Stage 2: The final, minimal production runtime image
FROM debian:trixie-slim

WORKDIR /app

# Copy only the compiled test-launch executable from the builder stage
COPY --from=builder /usr/src/gst-rtsp-server/build/examples/test-launch /usr/local/bin/test-launch

# Copy only the necessary runtime libraries from the builder stage
COPY --from=builder /usr/lib/x86_64-linux-gnu/libgstrtspserver-1.0.so.0* /usr/lib/x86_64-linux-gnu/

# Install ONLY the required runtime GStreamer packages for the final image
RUN apt-get update && apt-get install -y \
    libgstreamer1.0-0 gstreamer1.0-plugins-base gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad \
    && rm -rf /var/lib/apt/lists/*

# Expose  the default RTSP port
EXPOSE 8554

# Command to run the fully functional RTSP server using the video file
# The pipeline is provided as an argument to test-launch
# CMD ["/usr/local/bin/test-launch", "filesrc location=/app/video.mp4 ! qtdemux name=demux demux.video_0 ! h264parse ! rtph264pay name=pay0 pt=96 ! queue ! rtspclientsink location=rtsp://127.0.0.1:8554/camera1"]
CMD ["/usr/local/bin/test-launch", "( filesrc location=/app/video.mp4 ! qtdemux ! h264parse config-interval=-1 ! rtph264pay name=pay0 pt=96 )"]