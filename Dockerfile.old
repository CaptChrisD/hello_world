# ---- Build Stage ----
FROM elixir:1.10-alpine AS build

# Set environment variables for building the application
ENV MIX_ENV=prod \
      TEST=1 \
      LANG=C.UTF-8

# Install hex and rebar
RUN mix local.hex --force && \
      mix local.rebar --force

# Create the application build directory
RUN mkdir /app
WORKDIR /app

# Copy over all the necessary application files and directories
COPY config ./config
COPY lib ./lib
COPY priv ./priv
COPY mix.exs .
COPY mix.lock .

RUN ls

# Fetch the application dependencies and build the application
RUN mix deps.get
RUN mix deps.compile
RUN mix phx.digest
RUN mix release

# ---- Application Stage ----
FROM alpine AS app

ENV MIX_ENV=prod \
      LANG=C.UTF-8

# Install openssl
RUN apk update && \
      apk add --no-cache \
      bash \
      openssl-dev

# Copy over the build artifact from the previous step and create a non root user
WORKDIR /opt/app
COPY --from=build /app/_build .
RUN chown -R nobody: /opt/app
USER nobody

# Run the Phoenix app
CMD ["./prod/rel/hello_world/bin/hello_world", "start"]
