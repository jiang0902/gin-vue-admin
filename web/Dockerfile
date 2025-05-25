# 如果需要用 cicd ，请设置环境变量：
# variables:
#     DOCKER_BUILDKIT: 1

# FROM node:20-slim AS base
# ENV PNPM_HOME="/pnpm"
# ENV PATH="$PNPM_HOME:$PATH"
# RUN corepack enable
# COPY . /app
# WORKDIR /app


# FROM base AS prod-deps
# RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --prod

# FROM base AS build
# COPY --from=prod-deps /app/node_modules /app/node_modules
# RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install && pnpm run build


# FROM nginx:alpine
# LABEL MAINTAINER="bypanghu@163.com"
# COPY --from=base  /app/.docker-compose/nginx/conf.d/my.conf /etc/nginx/conf.d/my.conf
# COPY --from=build  /app/dist /usr/share/nginx/html
# RUN ls -al /usr/share/nginx/html

# 启用 BuildKit 并设置国内镜像源参数
# 在 CI/CD 中可注入参数：--build-arg REGISTRY_MIRROR=mirror.ccs.tencentyun.com
ARG REGISTRY_MIRROR=registry.cn-hangzhou.aliyuncs.com
ARG NPM_MIRROR=https://registry.npmmirror.com

# 阶段1: 基础环境构建
FROM ${REGISTRY_MIRROR}/library/node:20-slim AS base
ENV PNPM_HOME="/pnpm" \
    PNPM_REGISTRY=${NPM_MIRROR}  # 强制pnpm使用国内源
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable && \
    # 可选：设置系统级npm镜像源
    npm config set registry ${NPM_MIRROR} --global
COPY . /app
WORKDIR /app

# 阶段2: 生产依赖安装（利用缓存）
FROM base AS prod-deps
RUN --mount=type=cache,id=pnpm,target=/pnpm/store \
    pnpm install --prod --frozen-lockfile

# 阶段3: 完整构建阶段
FROM base AS build
COPY --from=prod-deps /app/node_modules /app/node_modules
RUN --mount=type=cache,id=pnpm,target=/pnpm/store \
    pnpm install && \
    pnpm run build

# 阶段4: 最终镜像（使用国内nginx镜像）
FROM ${REGISTRY_MIRROR}/library/nginx:alpine
LABEL MAINTAINER="bypanghu@163.com"
# 复制Nginx配置（推荐使用校验机制）
COPY --from=base /app/.docker-compose/nginx/conf.d/my.conf /etc/nginx/conf.d/
# 复制构建产物并验证
COPY --from=build /app/dist /usr/share/nginx/html
RUN ls -al /usr/share/nginx/html && \
    nginx -t  # 验证配置有效性
