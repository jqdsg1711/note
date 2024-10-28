# 部署指南

## mysql

1.seata_account.sql

```
/*
 Navicat Premium Data Transfer

 Source Server         : jq_dsg
 Source Server Type    : MySQL
 Source Server Version : 50742 (5.7.42-log)
 Source Host           : localhost:3306
 Source Schema         : seata_account

 Target Server Type    : MySQL
 Target Server Version : 50742 (5.7.42-log)
 File Encoding         : 65001

 Date: 16/07/2024 16:30:52
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for t_account
-- ----------------------------
DROP TABLE IF EXISTS `t_account`;
CREATE TABLE `t_account`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(11) NULL DEFAULT NULL COMMENT '用户id',
  `total` decimal(10, 0) NULL DEFAULT NULL COMMENT '总额度',
  `used` decimal(10, 0) NULL DEFAULT NULL COMMENT '已用额度',
  `residue` decimal(10, 0) NULL DEFAULT 0 COMMENT '剩余可用额度',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 2 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_account
-- ----------------------------
INSERT INTO `t_account` VALUES (1, 1, 1000, 700, 300);

-- ----------------------------
-- Table structure for undo_log
-- ----------------------------
DROP TABLE IF EXISTS `undo_log`;
CREATE TABLE `undo_log`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `context` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE INDEX `ux_undo_log`(`xid`, `branch_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of undo_log
-- ----------------------------

SET FOREIGN_KEY_CHECKS = 1;

```

2.seata_order.sql

```
/*
 Navicat Premium Data Transfer

 Source Server         : jq_dsg
 Source Server Type    : MySQL
 Source Server Version : 50742 (5.7.42-log)
 Source Host           : localhost:3306
 Source Schema         : seata_order

 Target Server Type    : MySQL
 Target Server Version : 50742 (5.7.42-log)
 File Encoding         : 65001

 Date: 16/07/2024 16:31:00
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for t_order
-- ----------------------------
DROP TABLE IF EXISTS `t_order`;
CREATE TABLE `t_order`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(11) NULL DEFAULT NULL COMMENT '用户id',
  `product_id` bigint(11) NULL DEFAULT NULL COMMENT '产品id',
  `count` int(11) NULL DEFAULT NULL COMMENT '数量',
  `money` decimal(11, 0) NULL DEFAULT NULL COMMENT '金额',
  `status` int(1) NULL DEFAULT NULL COMMENT '订单状态：0创建中，1已完结',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 20 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_order
-- ----------------------------
INSERT INTO `t_order` VALUES (14, 1, 1, 10, 100, 1);
INSERT INTO `t_order` VALUES (15, 1, 1, 10, 100, 1);
INSERT INTO `t_order` VALUES (16, 1, 1, 10, 100, 1);
INSERT INTO `t_order` VALUES (17, 1, 1, 10, 100, 1);
INSERT INTO `t_order` VALUES (18, 1, 1, 10, 100, 1);
INSERT INTO `t_order` VALUES (19, 1, 1, 10, 100, 1);

-- ----------------------------
-- Table structure for undo_log
-- ----------------------------
DROP TABLE IF EXISTS `undo_log`;
CREATE TABLE `undo_log`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `context` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE INDEX `ux_undo_log`(`xid`, `branch_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of undo_log
-- ----------------------------

SET FOREIGN_KEY_CHECKS = 1;

```

3.seata_storage.sql

```
/*
 Navicat Premium Data Transfer

 Source Server         : jq_dsg
 Source Server Type    : MySQL
 Source Server Version : 50742 (5.7.42-log)
 Source Host           : localhost:3306
 Source Schema         : seata_storage

 Target Server Type    : MySQL
 Target Server Version : 50742 (5.7.42-log)
 File Encoding         : 65001

 Date: 16/07/2024 16:31:17
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for t_storage
-- ----------------------------
DROP TABLE IF EXISTS `t_storage`;
CREATE TABLE `t_storage`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `product_id` bigint(11) NULL DEFAULT NULL COMMENT '产品id',
  `total` int(11) NULL DEFAULT NULL COMMENT '总库存',
  `used` int(11) NULL DEFAULT NULL COMMENT '已用库存',
  `residue` int(11) NULL DEFAULT NULL COMMENT '剩余库存',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 2 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_storage
-- ----------------------------
INSERT INTO `t_storage` VALUES (1, 1, 100000, 60, 99940);

-- ----------------------------
-- Table structure for undo_log
-- ----------------------------
DROP TABLE IF EXISTS `undo_log`;
CREATE TABLE `undo_log`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `context` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE INDEX `ux_undo_log`(`xid`, `branch_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of undo_log
-- ----------------------------

SET FOREIGN_KEY_CHECKS = 1;

```

## docker-stack.yml

```
version: '3.7'

services:
  order:
    image: order
    ports:
      - "3001:3001"  # 主应用程序的端口
      - "8081:8081"  # Actuator 暴露 Prometheus 指标的端口
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: '0.50'
          memory: '512M'
        reservations:
          cpus: '0.25'
          memory: '256M'

  account:
    image: account
    ports:
      - "3002:3002"  # 主应用程序的端口
      - "8082:8082"  # Actuator 暴露 Prometheus 指标的端口
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: '0.50'
          memory: '512M'
        reservations:
          cpus: '0.25'
          memory: '256M'

  storage:
    image: storage
    ports:
      - "3003:3003"  # 主应用程序的端口
      - "8083:8083"  # Actuator 暴露 Prometheus 指标的端口
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: '0.50'
          memory: '512M'
        reservations:
          cpus: '0.25'
          memory: '256M'
```

prometheus.yml

```
global:
  scrape_interval: 1s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'docker01'
    static_configs:
      - targets: ['192.168.88.129:9323']  # cadvisor on docker01
  - job_name: 'docker02'
    static_configs:
      - targets: ['192.168.88.128:9323']  # cadvisor on docker02
  - job_name: 'docker03'
    static_configs:
      - targets: ['192.168.88.127:9323']  # cadvisor on docker03
  - job_name: 'order'
    static_configs:
      - targets: ['192.168.88.129:8081', '192.168.88.128:8081', '192.168.88.127:8081']  # 配置 order 服务实例的 Actuator 端点
    metrics_path: '/actuator/prometheus'  # 抓取 Prometheus 指标的路径
  - job_name: 'account'
    static_configs:
      - targets: ['192.168.88.129:8082', '192.168.88.128:8082', '192.168.88.127:8082']  # 配置 account 服务实例的 Actuator 端点
    metrics_path: '/actuator/prometheus'  # 抓取 Prometheus 指标的路径
  - job_name: 'storage'
    static_configs:
      - targets: ['192.168.88.129:8083', '192.168.88.128:8083', '192.168.88.127:8083']  # 配置 storage 服务实例的 Actuator 端点
    metrics_path: '/actuator/prometheus'  # 抓取 Prometheus 指标的路径



#  
#  - job_name: 'spring-boot-microservices'
#    metrics_path: '/actuator/prometheus'
#    static_configs:
#      - targets:
#        - '192.168.88.129:3001'
#        - '192.168.88.128:3001'
#        - '192.168.88.127:3001'    
#  - job_name: 'spring-boot-microservices'
#    metrics_path: '/actuator/prometheus'
#    static_configs:
#      - targets:
#        - 'docker01:3001'
#        - 'docker02:3001'
#        - 'docker03:3001'
#        - 'docker01:3002'
#        - 'docker02:3002'
#        - 'docker03:3002'
#        - 'docker01:3003'
#        - 'docker02:3003'
#        - 'docker03:3003'
#  - job_name: 'order_service'
#    metrics_path: '/actuator/prometheus'
#    static_configs:
#      - targets: ['100.64.225.239:3001']  # 替换为实际的服务名称和端口
#
#  - job_name: 'account_service'
#    metrics_path: '/actuator/prometheus'
#    static_configs:
#      - targets: ['100.64.225.239:3002']  # 替换为实际的服务名称和端口
#
#  - job_name: 'storage_service'
#    metrics_path: '/actuator/prometheus'
#    static_configs:
#      - targets: ['100.64.225.239:3003']  # 替换为实际的服务名称和端口
```