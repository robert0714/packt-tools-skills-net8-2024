# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0

FROM --platform=${BUILDPLATFORM} mcr.microsoft.com/dotnet/sdk:8.0 AS builder
ARG TARGETARCH
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["Chapter05/Northwind.WebApi", "/src/Chapter05/Northwind.WebApi/"]
COPY ["Chapter01", "/src/Chapter01"]
COPY ["Chapter05/OTEL-v2/pb/demo.proto", "/src/Chapter05/Northwind.WebApi/src/protos/demo.proto"]
RUN dotnet restore "/src/Chapter05/Northwind.WebApi/Northwind.WebApi.csproj" -r linux-$TARGETARCH

WORKDIR "/src/Chapter05/Northwind.WebApi"
# RUN dotnet remove package OpenTelemetry.Exporter.Console
# RUN dotnet remove package OpenTelemetry.Extensions.Hosting

# RUN dotnet remove package OpenTelemetry.Instrumentation.AspNetCore
# RUN dotnet remove package OpenTelemetry.Instrumentation.EntityFrameworkCore
# RUN dotnet remove package OpenTelemetry.Instrumentation.SqlClient

RUN dotnet tool install -g dotnet-grpc
ENV PATH="$PATH:/root/.dotnet/tools"
RUN dotnet-grpc add-file "./src/protos/demo.proto" --services None

RUN dotnet add package Google.Protobuf --version 3.31.1
RUN dotnet add package Grpc.Tools --version 2.68.1
RUN dotnet add package OpenTelemetry.AutoInstrumentation --version 1.11.0

# Insert <DockerDefaultTargetOS>Linux</DockerDefaultTargetOS>
RUN grep -q "<DockerDefaultTargetOS>" /src/Chapter05/Northwind.WebApi/Northwind.WebApi.csproj || \
    sed -i '/<\/PropertyGroup>/i \    <DockerDefaultTargetOS>Linux<\/DockerDefaultTargetOS>' /src/Chapter05/Northwind.WebApi/Northwind.WebApi.csproj

# Restore as distinct layers
WORKDIR /src
RUN dotnet restore "/src/Chapter05/Northwind.WebApi/Northwind.WebApi.csproj" -r linux-$TARGETARCH

# Build and publish a release
WORKDIR "/src/Chapter05/Northwind.WebApi"
RUN dotnet build "./Northwind.WebApi.csproj" -r linux-$TARGETARCH -c $BUILD_CONFIGURATION -o /app/build

# -----------------------------------------------------------------------------

FROM builder AS publish
ARG TARGETARCH
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./Northwind.WebApi.csproj" -r linux-$TARGETARCH -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# -----------------------------------------------------------------------------

FROM mcr.microsoft.com/dotnet/aspnet:8.0
USER app
WORKDIR /app 
COPY --from=publish /app/publish .

# Copy the published application and OTEL binaries
USER app
WORKDIR /app
COPY --from=publish /app/publish . 

USER app
WORKDIR /app
COPY --from=publish /app/publish .
COPY ["Chapter05/OTEL-v2-auto-instrumentation/instrument.sh", "/app/instrument.sh"]

USER root
RUN mkdir -p "/var/log/opentelemetry/dotnet"
RUN chown app "/var/log/opentelemetry/dotnet"
RUN chown app "/app/instrument.sh"

# install curl, wget
RUN apt-get update && apt-get install -y curl wget && rm -rf /var/lib/apt/lists/*

USER app
 
ENV OTEL_DOTNET_AUTO_TRACES_ADDITIONAL_SOURCES=Accounting.Consumer


ENTRYPOINT ["./instrument.sh", "dotnet", "Northwind.WebApi.dll"] 
