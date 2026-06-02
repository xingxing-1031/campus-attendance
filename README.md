# 校园智能考勤系统

## 项目概述

校园智能考勤微信小程序，支持学生GPS定位打卡与扫码打卡，配备人脸识别核验与活体检测，教师可创建考勤任务并实时查看签到情况，管理员可管理师生账号、班级信息并汇总全校考勤数据。

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | 微信小程序原生（WXML + WXSS + JS） |
| 后端 | Java SpringBoot 2.7 + MyBatis-Plus |
| 数据库 | MySQL 8.0 |
| 认证 | JWT Token + BCrypt 密码加密 |
| 工具 | Hutool、Apache POI、Druid 连接池 |

## 项目结构

```
solo-kaifa/
├── campus-attendance-server/          # SpringBoot后端
│   ├── pom.xml                        # Maven依赖配置
│   └── src/main/
│       ├── java/com/campus/attendance/
│       │   ├── CampusAttendanceApplication.java   # 启动类
│       │   ├── entity/                # 实体类
│       │   │   ├── User.java
│       │   │   ├── ClassInfo.java
│       │   │   ├── AttendanceTask.java
│       │   │   ├── AttendanceRecord.java
│       │   │   └── LeaveApplication.java
│       │   ├── dao/                   # 数据访问层（MyBatis-Plus Mapper）
│       │   │   ├── UserDao.java
│       │   │   ├── ClassInfoDao.java
│       │   │   ├── AttendanceTaskDao.java
│       │   │   ├── AttendanceRecordDao.java
│       │   │   └── LeaveApplicationDao.java
│       │   ├── service/               # 业务逻辑层
│       │   │   ├── UserService.java
│       │   │   ├── AttendanceService.java
│       │   │   ├── LeaveService.java
│       │   │   ├── AdminService.java
│       │   │   └── ScheduledTaskService.java
│       │   ├── controller/            # 接口控制层
│       │   │   ├── AuthController.java
│       │   │   ├── StudentController.java
│       │   │   ├── TeacherController.java
│       │   │   ├── AdminController.java
│       │   │   └── GlobalExceptionHandler.java
│       │   ├── dto/                   # 数据传输对象
│       │   ├── config/                # 配置类
│       │   ├── util/                  # 工具类
│       │   └── interceptor/           # 拦截器
│       └── resources/
│           ├── application.yml        # 应用配置
│           └── db/init.sql            # 数据库初始化脚本
│
├── campus-attendance-miniapp/         # 微信小程序前端
│   ├── app.js / app.json / app.wxss   # 应用入口
│   ├── project.config.json
│   ├── utils/                         # 工具模块
│   │   ├── api.js                     # 网络请求封装
│   │   ├── util.js                    # 通用函数
│   │   └── auth.js                    # 权限与设备能力
│   ├── components/navbar/             # 自定义导航栏组件
│   └── pages/
│       ├── login/                     # 登录页
│       ├── student/
│       │   ├── checkin/               # 学生打卡
│       │   ├── records/               # 考勤记录
│       │   └── leave/                 # 请假申请
│       ├── teacher/
│       │   ├── task/                  # 任务管理
│       │   ├── check/                 # 签到详情
│       │   └── leave-approval/        # 请假审批
│       └── admin/
│           ├── users/                 # 用户管理
│           ├── classes/               # 班级管理
│           └── statistics/            # 数据统计
│
└── README.md                          # 本文件
```

## 数据库部署

### 1. 安装 MySQL 8.0+

### 2. 执行初始化脚本

```bash
mysql -u root -p < campus-attendance-server/src/main/resources/db/init.sql
```

或登录 MySQL 后执行：

```sql
source campus-attendance-server/src/main/resources/db/init.sql;
```

### 3. 数据库表结构

| 表名 | 说明 | 核心字段 |
|------|------|----------|
| `user` | 用户表 | user_no, password(BCrypt), role(1学生2教师3管理员), class_id |
| `class_info` | 班级信息表 | class_name, grade, major, teacher_id |
| `attendance_task` | 考勤任务表 | check_type(GPS/二维码), need_face, start_time, end_time, qr_code |
| `attendance_record` | 考勤记录表 | task_id, student_id, face_verified, face_score, status |
| `leave_application` | 请假申请表 | leave_type(病假/事假), status(待审批/已批准/已驳回) |

### 4. 预置测试账号

| 角色 | 账号 | 密码 |
|------|------|------|
| 管理员 | admin001 | admin123 |
| 教师 | t001 | teacher123 |
| 学生 | s001 | student123 |

## 后端部署

### 环境要求

- JDK 1.8+
- Maven 3.6+
- MySQL 8.0+

### 配置修改

编辑 `campus-attendance-server/src/main/resources/application.yml`，修改数据库连接信息：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/campus_attendance?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root      # 修改为你的数据库用户名
    password: root      # 修改为你的数据库密码
```

### 启动方式

#### 方式一：Maven 命令行

```bash
cd campus-attendance-server
mvn clean package -DskipTests
java -jar target/campus-attendance-1.0.0.jar
```

#### 方式二：IDE 导入运行（推荐 IntelliJ IDEA）

1. 打开 IntelliJ IDEA
2. File → Open → 选择 `campus-attendance-server` 目录（包含 pom.xml）
3. IDEA 自动识别 Maven 项目，等待依赖下载完成
4. 找到 `CampusAttendanceApplication.java`
5. 右键 → Run 'CampusAttendanceApplication'
6. 服务启动在 `http://localhost:8080`

### API 接口预览

| 模块 | 路径前缀 | 说明 |
|------|----------|------|
| 认证 | `/api/auth/*` | 登录、密码重置、获取用户信息 |
| 学生端 | `/api/student/*` | 查看任务、打卡签到、考勤记录、请假 |
| 教师端 | `/api/teacher/*` | 创建任务、查看签到、审批请假、导出数据 |
| 管理员端 | `/api/admin/*` | 用户管理、班级管理、统计汇总 |

## 微信小程序部署

### 环境准备

1. 下载并安装 [微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)
2. 注册微信小程序账号，获取 AppID

### 导入项目

1. 打开微信开发者工具
2. 选择「导入项目」
3. 项目目录选择 `campus-attendance-miniapp`
4. AppID 填写你的小程序 AppID（开发阶段可使用测试号）
5. 点击「导入」

### 配置修改

编辑 `campus-attendance-miniapp/utils/api.js` 中的 `baseUrl`：

```javascript
const baseUrl = app ? app.globalData.baseUrl : 'http://localhost:8080';
// 如果后端部署在服务器，修改为实际IP或域名
// 例如: 'https://api.yourdomain.com'
```

### 开发调试

1. 在微信开发者工具中，点击「编译」即可预览
2. 如需真机调试，点击「预览」生成二维码，用手机微信扫码
3. 注意：真机调试时 `localhost` 无法访问，需将后端部署到公网或使用内网穿透工具

### 权限说明

小程序需要以下权限：

| 权限 | 用途 |
|------|------|
| 相机 (scope.camera) | 人脸拍照验证 |
| 定位 (scope.userLocation) | GPS打卡定位 |
| 相册 (scope.writePhotosAlbum) | 保存考勤数据（可选） |

## 人脸识别对接说明

当前人脸验证功能为模拟实现，生产环境需对接第三方人脸识别API。

### 推荐对接平台

| 平台 | 文档地址 | 特点 |
|------|----------|------|
| 百度AI人脸识别 | https://ai.baidu.com/tech/face | 支持活体检测、人脸比对 |
| 腾讯云人脸识别 | https://cloud.tencent.com/product/facerecognition | 微信生态兼容好 |
| 阿里云人脸识别 | https://www.aliyun.com/product/facebody | 接口稳定 |

### 对接步骤

1. 注册对应平台账号，获取 API Key 和 Secret
2. 修改 `application.yml` 中 `face.verify` 配置
3. 修改 `FaceVerifyUtil.java` 中的 `compareFace` 和 `livenessDetect` 方法
4. 调用真实API进行人脸比对和活体检测

### 人脸数据安全说明

- 人脸图片仅用于临时比对校验，不保存原始图片到服务器
- 比对完成后立即释放 Base64 数据
- 仅记录验证结果（通过/失败）和比对分数

## 功能清单

### 学生端
- [x] GPS定位打卡
- [x] 扫码打卡
- [x] 人脸识别核验（含活体检测）
- [x] 防照片/视频代打卡
- [x] 查看当日考勤状态
- [x] 历史考勤记录
- [x] 请假申请提交
- [x] 查看审批结果

### 教师端
- [x] 创建考勤任务（GPS/二维码/两者均可）
- [x] 设置打卡时长
- [x] 生成签到二维码
- [x] 实时查看班级签到情况
- [x] 请假审批（批准/驳回）
- [x] 导出考勤数据表（CSV格式）
- [x] 人脸验证开关控制

### 管理员端
- [x] 师生账号管理（增删改查/启用禁用）
- [x] 班级与课程信息管理
- [x] 全校考勤数据汇总
- [x] 缺勤情况统计
- [x] 异常考勤记录核查
- [x] 近7天考勤趋势

### 附加功能
- [x] 考勤超时自动结束签到（每60秒定时扫描）
- [x] 定位+人脸双重校验防代打卡
- [x] 自动申请相机、定位权限
- [x] 数据云端存储 + 本地缓存（Token/用户信息）
- [x] 界面适配移动端，简约校园风格
- [x] JWT Token 认证，BCrypt密码加密

## 注意事项

1. 生产环境部署前，请修改 `JwtUtil.java` 中的 `SECRET` 密钥
2. 生产环境建议关闭 MyBatis SQL 日志输出
3. 微信小程序正式发布需通过微信审核
4. 人脸识别功能需申请相应API权限后方可正式使用
5. 数据库密码请使用强密码，并定期更换
