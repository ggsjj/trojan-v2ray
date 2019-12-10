# trojan-v2ray
手工安装




v2ray 要先改时区

cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

或者

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

cnetos7 换内核

rpm -ivh http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force --nodeps

个别机子安装成功 没有切换成功 看/etc/grub.conf 

default=1  #1为顺序

安装Nginx
1、添加源

　　默认情况Centos7中无Nginx的源，最近发现Nginx官网提供了Centos的源地址。因此可以如下执行命令添加源：

sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
 

2、安装Nginx

　　通过yum search nginx看看是否已经添加源成功。如果成功则执行下列命令安装Nginx。

sudo yum install -y nginx
 

3、启动Nginx并设置开机自动运行

systemctl start nginx.service
systemctl enable nginx.service



systemctl restart nginx.service   重启NG

```bash

1、firewalld的基本使用
启动： systemctl start firewalld
关闭： systemctl stop firewalld
查看状态： systemctl status firewalld 
开机禁用  ： systemctl disable firewalld
开机启用  ： systemctl enable firewalld
```

=================================================
```bash

curl  https://get.acme.sh | sh
alias acme.sh=~/.acme.sh/acme.sh
acme.sh --issue -d en.vvppmm.xyz -w /home/en.vvppmm.xyz

acme.sh --issue -d en.vvppmm.xyz -w /home/en.vvppmm.xyz --standalone -k ec-256



<

acme.sh --list
 
# 删除证书
acme.sh remove <SAN_Domains>
>不重要
```

```bash
80.conf


server
    {
        listen 80;

        server_name en.vvppmm.xyz;
        index index.html index.htm;
        root  /home/en.vvppmm.xyz;



        location ~ /.well-known {
            allow all;
        }

        location ~ /\.
        {
            deny all;
        }

    }


```

```bash

443.conf

server {
        listen  443 ssl http2;
        server_name en.vvppmm.xyz;

  ssl_certificate /root/.acme.sh/en.vvppmm.xyz/fullchain.cer;
  ssl_certificate_key /root/.acme.sh/en.vvppmm.xyz/en.vvppmm.xyz.key;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers HIGH:!aNULL:!MD5;

        location / {
        proxy_pass https://www.asmag.com;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        }
        location /api {　　　#路径
        proxy_pass http://localhost:2333;　　#V2ray端口
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        }
    }
	
```

```bash	
官方配置方式 nginx

server {
  listen 443 ssl;
  ssl on;
  ssl_certificate /root/.acme.sh/en.vvppmm.xyz/fullchain.cer;
  ssl_certificate_key /root/.acme.sh/en.vvppmm.xyz/en.vvppmm.xyz.key;
  ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers           HIGH:!aNULL:!MD5;
  server_name           en.vvppmm.xyz;

    location / {
      proxy_redirect off;
      proxy_pass https://www.asmag.com;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";


    }


    location /api {
      proxy_redirect off;
      proxy_pass http://127.0.0.1:54123;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

  #add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}	
```


```bash
	
80 443同一个配置 官方配置方式

	


server {
  listen 80;
  listen 443 ssl;
  server_name en.vvppmm.xyz;
  root  /home/en.vvppmm.xyz;



  ssl_certificate /root/.acme.sh/en.vvppmm.xyz/fullchain.cer;
  ssl_certificate_key /root/.acme.sh/en.vvppmm.xyz/en.vvppmm.xyz.key;
  ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers           HIGH:!aNULL:!MD5;

    if ($server_port !~ 443){
        rewrite ^(/.*)$ https://$host$1 permanent;
    }



    location / {
      proxy_redirect off;
      proxy_pass https://www.asmag.com;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";


    }


    location /api {
      proxy_redirect off;
      proxy_pass http://127.0.0.1:54123;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

  #add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    location ~ /.well-known {
            allow all;
    }

#可访问文本不需要可以删除
    location ~ .*\.(txt)$   
    {
        expires      30d;

    }
     
	 location ~ /\.
    {
            deny all;
     }

 
}
	
```

```bash
	=========caddy=============
	
	如不用NGINX申请证书
	关闭防火墙或开放端口
	
iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -m state --state NEW -m udp -p udp --dport 443 -j ACCEPT
 
# 删除防火墙规则，内容一样把 -I 换成 -D 就行了：
iptables -D INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
iptables -D INPUT -m state --state NEW -m udp -p udp --dport 443 -j ACCEPT
	
```

```bash	
	CADDY日 志：/tmp/caddy.log
	
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/caddy_install.sh && chmod +x caddy_install.sh && bash caddy_install.sh
	

vi /usr/local/caddy/Caddyfile

启动caddy service caddy start
重启caddy service caddy restart
查看状态 service caddy status

启动：/etc/init.d/caddy start
停止：/etc/init.d/caddy stop
重启：/etc/init.d/caddy restart
查看状态：/etc/init.d/caddy status
查看Caddy启动日志：tail -f /tmp/caddy.log
安装目录：/usr/local/caddy
Caddy配置文件位置：/usr/local/caddy/Caddyfile
Caddy自动申请SSL证书位置：/.caddy/acme/acme-v01.api.letsencrypt.org/sites/xxx.xxx(域名)/

```

```bash
用233脚本 CADDY配置
	
cl.vvppmm.xyz {
    tls com388488@gmail.com
    gzip
timeouts none
    proxy / http://majuredata.com {
        without /api
    }
    proxy /api 127.0.0.1:54123 {
        without /api
        websocket
    }
}
```

```bash
	v2ray推官方CADDY配置
	
vip.vvppmm.xyz
{
    tls xhhhdkk@gmail.com
    gzip
timeouts none
    proxy / http://majuredata.com {

    }
    proxy /api 127.0.0.1:54123 {

        websocket
	header_upstream -Origin
    }
}

```

```bash

=============================锐速==================

yum install -y net-tools

wget --no-check-certificate -qO /tmp/appex.sh "https://raw.githubusercontent.com/0oVicero0/serverSpeeder_Install/master/appex.sh" && bash /tmp/appex.sh 'install'

/appex/bin/serverSpeeder.sh status

====================================
```

```bash
	
	bash <(curl -L -s https://install.direct/go.sh)

此脚本会自动安装以下文件：

    /usr/bin/v2ray/v2ray：V2Ray 程序；
    /usr/bin/v2ray/v2ctl：V2Ray 工具；
    /etc/v2ray/config.json：配置文件；
    /usr/bin/v2ray/geoip.dat：IP 数据文件
    /usr/bin/v2ray/geosite.dat：域名数据文件

	脚本运行完成后，你需要：

    编辑 /etc/v2ray/config.json 文件来配置你需要的代理方式；
    运行 service v2ray start 来启动 V2Ray 进程；
    之后可以使用 service v2ray start|stop|status|reload|restart|force-reload 控制 V2Ray 的运行。

	
```



```bash
	
	
	v2ray服务配置文件
	
	
	
	
	{
	"log": {
		"access": "/var/log/v2ray/access.log",
		"error": "/var/log/v2ray/error.log",
		"loglevel": "warning"
	},
	"inbounds": [
		{
			"port": 54123, #V2ray端口
			"protocol": "vmess",
			"settings": {
				"clients": [
					{
						"id": "eefebf6c-b6e9-4ea3-a4c3-bee890d30232",  #UUID
						"level": 1,
						"alterId": 254
					}
				]
			},
			"streamSettings": {
				"network": "ws",
    "wsSettings": {
      "path": "/api"    #路径
      }
			}
		}

	],
	"outbounds": [
		{
			"protocol": "freedom",
			"settings": {}
		},
		{
			"protocol": "blackhole",
			"settings": {},
			"tag": "blocked"
		},
		{
			"protocol": "freedom",
			"settings": {},
			"tag": "direct"
		},
		{
			"protocol": "mtproto",
			"settings": {},
			"tag": "tg-out"
		}
		//include_out_config
		//
	],
	"dns": {
		"server": [
			"1.1.1.1",
			"1.0.0.1",
			"8.8.8.8",
			"8.8.4.4",
			"localhost"
		]
	},
	"routing": {
		"domainStrategy": "IPOnDemand",
		"rules": [
			{
				"type": "field",
				"ip": [
					"0.0.0.0/8",
					"10.0.0.0/8",
					"100.64.0.0/10",
					"127.0.0.0/8",
					"169.254.0.0/16",
					"172.16.0.0/12",
					"192.0.0.0/24",
					"192.0.2.0/24",
					"192.168.0.0/16",
					"198.18.0.0/15",
					"198.51.100.0/24",
					"203.0.113.0/24",
					"::1/128",
					"fc00::/7",
					"fe80::/10"
				],
				"outboundTag": "blocked"
			},
			{
				"type": "field",
				"inboundTag": ["tg-in"],
				"outboundTag": "tg-out"
			}
			,
			{
				"type": "field",
				"domain": [
					"domain:epochtimes.co"
				],
				"outboundTag": "blocked"
			}			,
                {
                    "type": "field",
                    "protocol": [
                        "bittorrent"
                    ],
                    "outboundTag": "blocked"
                }
			//include_ban_ad
			//include_rules
			//
		]
	},
	"transport": {
		"kcpSettings": {
            "uplinkCapacity": 100,
            "downlinkCapacity": 100,
            "congestion": true
        },
		"sockopt": {
			"tcpFastOpen": true
		}
	}
}
	
	
	
```

```bash
	
	
	
	
	==========
	
	
	
	
	
	
	
{
	"log": {
		"access": "/var/log/v2ray/access.log",
		"error": "/var/log/v2ray/error.log",
		"loglevel": "warning"
	},
	"inbounds": [
		{
			"port": 54123,
			"protocol": "vmess",
			"settings": {
				"clients": [
					{
						"id": "415fe30d-198e-4a5c-8491-4c79c8aec3f1",
						"level": 1,
						"alterId": 224
					},     # 注意要有逗号
					{
						"id": "a3131d41-637a-495b-8d2d-375ff6c6d3cb",  #多用户?谡饫锒嘈?馊?注意逗号
						"level": 1,
						"alterId": 225
					}
				]
			},
            "streamSettings": {
                "network": "ws",
                
                "tlsSettings": {
                    "serverName": "w2w.vvppmm.xyz"
                },
                "wsSettings": {
				"security": "tls",  #不知有没有作用
                    "path": "/api",
                    "headers": {
                        "Host": "w2w.vvppmm.xyz"
                    }
                }
            }

		}

	],
	"outbounds": [
		{
			"protocol": "freedom",
			"settings": {}
		},
		{
			"protocol": "blackhole",
			"settings": {},
			"tag": "blocked"
		},
		{
			"protocol": "freedom",
			"settings": {},
			"tag": "direct"
		},
		{
			"protocol": "mtproto",
			"settings": {},
			"tag": "tg-out"
		}
		//include_out_config
		//
	],
	"dns": {
		"server": [
			"1.1.1.1",
			"1.0.0.1",
			"8.8.8.8",
			"8.8.4.4",
			"localhost"
		]
	},
	"routing": {
		"domainStrategy": "IPOnDemand",
		"rules": [
			{
				"type": "field",
				"ip": [
					"0.0.0.0/8",
					"10.0.0.0/8",
					"100.64.0.0/10",
					"127.0.0.0/8",
					"169.254.0.0/16",
					"172.16.0.0/12",
					"192.0.0.0/24",
					"192.0.2.0/24",
					"192.168.0.0/16",
					"198.18.0.0/15",
					"198.51.100.0/24",
					"203.0.113.0/24",
					"::1/128",
					"fc00::/7",
					"fe80::/10"
				],
				"outboundTag": "blocked"
			},
			{
				"type": "field",
				"inboundTag": ["tg-in"],
				"outboundTag": "tg-out"
			}
			,
			{
				"type": "field",
				"domain": [
					"domain:epochtimes.co"
				],
				"outboundTag": "blocked"
			}			,
                {
                    "type": "field",
                    "protocol": [
                        "bittorrent"
                    ],
                    "outboundTag": "blocked"
                }
			//include_ban_ad
			//include_rules
			//
		]
	},
	"transport": {
		"kcpSettings": {
            "uplinkCapacity": 100,
            "downlinkCapacity": 100,
            "congestion": true
        },
		"sockopt": {
			"tcpFastOpen": true
		}
	}
}

```

```bash

======================修改自233blog配置的自建文件=============


{
	"log": {
		"access": "/var/log/v2ray/access.log",
		"error": "/var/log/v2ray/error.log",
		"loglevel": "warning"
	},
	"inbounds": [
		{
			"port": 54123,    #v2ray端口
			"protocol": "vmess",
			"settings": {
				"clients": [
					{
						"id": "415fe30d-198e-4a5c-8491-4c79c8aec3f1",
						"level": 1,
						"alterId": 224
					},  #当只有一个这里删除
					{
						"id": "a3131d41-637a-495b-8d2d-375ff6c6d3cb",   # 新增用户  如要删除去注意结尾逗号
						"level": 1,
						"alterId": 225
					},
					{
						"id": "0b006724-7e32-4dbc-94b8-8db0318cb64a",
						"level": 1,
						"alterId": 234
					}
				
				]
			},
            "streamSettings": {

                "network": "ws",

             "wsSettings": {
		       "security": "tls", 
                    "path": "/api"},   # /api  分流地址
 
		"sniffing": {
			"enabled": true,
			"destOverride": [
				"http",
				"tls"
				]
                }
            }

	   }
     #    ,
     #   {                                              #socks5 配置 不知会不会封
     #       "protocol": "socks",
     #       "port": 10086,
     #       "settings": {
     #           "auth": "password",
     #           "accounts": [
     #               {
     #                   "user": "user",
     #                   "pass": "qq123456"
     #               }
     #           ],
     #           "udp": true,
     #           "timeout": 0,
     #           "userLevel": 1
     #       }
     #   }


	],
	"outbounds": [
		{
			"protocol": "freedom",
			"settings": {}
		},
		{
			"protocol": "blackhole",
			"settings": {},
			"tag": "blocked"
		},
		{
			"protocol": "freedom",
			"settings": {},
			"tag": "direct"
		},
		{
			"protocol": "mtproto",
			"settings": {},
			"tag": "tg-out"
		}

	],
	"dns": {
		"server": [
			"1.1.1.1",
			"1.0.0.1",
			"8.8.8.8",
			"8.8.4.4",
			"localhost"
		]
	},
	"routing": {
		"domainStrategy": "IPOnDemand",
		"rules": [
			{
				"type": "field",
				"ip": [
					"0.0.0.0/8",
					"10.0.0.0/8",
					"100.64.0.0/10",
					"127.0.0.0/8",
					"169.254.0.0/16",
					"172.16.0.0/12",
					"192.0.0.0/24",
					"192.0.2.0/24",
					"192.168.0.0/16",
					"198.18.0.0/15",
					"198.51.100.0/24",
					"203.0.113.0/24",
					"::1/128",
					"fc00::/7",
					"fe80::/10"
				],
				"outboundTag": "blocked"
			},
			{
				"type": "field",
				"inboundTag": ["tg-in"],
				"outboundTag": "tg-out"
			}
			,
			{
				"type": "field",
				"domain": [
				"domain:aaaa.com",
				"domain:bbbb.com"  //屏蔽域名
				],
				"outboundTag": "blocked"
			},
               # {             禁 BT下载                  
               #     "type": "field",
               #     "protocol": [
               #         "bittorrent"
               #     ],
               #     "outboundTag": "blocked"
               # }
			//include_ban_ad
			//include_rules
			//
		]
	},
	"transport": {
		"kcpSettings": {
            "uplinkCapacity": 100,
            "downlinkCapacity": 100,
            "congestion": true
        },
		"sockopt": {
			"tcpFastOpen": true
		}
	}
}


```

```bash
=================mkcp============

{
	"log": {
		"access": "/var/log/v2ray/access.log",
		"error": "/var/log/v2ray/error.log",
		"loglevel": "warning"
	},
	"inbounds": [
		{
			"port": 54123,
			"protocol": "vmess",
			"settings": {
				"clients": [
					{
						"id": "23c518b6-f4f7-4e4b-bc68-b853cbf6e91e",
						"level": 1,
						"alterId": 233
					}
				]
			},
			"streamSettings": {
				"network": "mkcp",
				"kcpSettings": {
					"header": {
						"type": "wechat-video"
					}
				}
			},
			"sniffing": {
				"enabled": true,
				"destOverride": [
					"http",
					"tls"
				]
			}
		}

	],
	"outbounds": [
		{
			"protocol": "freedom",
			"settings": {}
		},
		{
			"protocol": "blackhole",
			"settings": {},
			"tag": "blocked"
        },
        {
			"protocol": "freedom",
			"settings": {},
			"tag": "direct"
		}
	],
	"dns": {
		"server": [
			"1.1.1.1",
			"1.0.0.1",
			"8.8.8.8",
			"8.8.4.4",
			"localhost"
		]
	},
	"routing": {
		"domainStrategy": "IPOnDemand",		
		"rules": [
			{
				"type": "field",
				"ip": [
					"0.0.0.0/8",
					"10.0.0.0/8",
					"100.64.0.0/10",
					"127.0.0.0/8",
					"169.254.0.0/16",
					"172.16.0.0/12",
					"192.0.0.0/24",
					"192.0.2.0/24",
					"192.168.0.0/16",
					"198.18.0.0/15",
					"198.51.100.0/24",
					"203.0.113.0/24",
					"::1/128",
					"fc00::/7",
					"fe80::/10"
				],
				"outboundTag": "blocked"
			},
			{
				"type": "field",
				"inboundTag": ["tg-in"],
				"outboundTag": "tg-out"
			}
			,
			{
				"type": "field",
				"domain": [
					"domain:bbb.om"
				],
				"outboundTag": "blocked"
			}			,
                {
                    "type": "field",
                    "protocol": [
                        "bittorrent"
                    ],
                    "outboundTag": "blocked"
                }

		]
	},
	"transport": {
		"kcpSettings": {
            "uplinkCapacity": 100,
            "downlinkCapacity": 100,
            "congestion": true
        },
		"sockopt": {
			"tcpFastOpen": true
		}
	}
}

```

======多端口多协议=========


```bash



{
	"log": {
		"access": "/var/log/v2ray/access.log",
		"error": "/var/log/v2ray/error.log",
		"loglevel": "warning"
	},
	
	"inbounds": [
	{
			"port": 54123, 
			"protocol": "vmess",
			"settings": {
				"clients": [
					{
						"id": "bd533185-d5d4-4cf9-a82e-43df4faffed3",
						"level": 1,
						"alterId": 168
					}       ]
			            },
            "streamSettings": {
                "network": "ws",
             "wsSettings": {
                    "path": "/gogapi"
					       } 
                              }
	},

		{
			"port": 5223,
			"protocol": "vmess",
			"settings": {
				"clients": [
					{
						"id": "23c518b6-f4f7-4e4b-bc68-b853cbf6e91e",
						"level": 1,
						"alterId": 233
					}
				          ]
			            },
			"streamSettings": {
				"network": "mkcp",
				"kcpSettings": {
					"header": {
						"type": "wechat-video"
					          }
				             }
		                       }
		
			
		},    #多端口  多用户在"inbounds": [ {ws},{kcp},{ws} ]  ws:配置内容
		
		   {
			"port": 5224,
			"protocol": "vmess",
			"settings": {
				"clients": [
					{
						"id": "23c518b6-f4f7-4e4b-bc68-b853cbf6e91e",
						"level": 1,
						"alterId": 233
					}
				          ]
			            },
			"streamSettings": {
				"network": "mkcp",
				"kcpSettings": {
					"header": {
						"type": "wechat-video"
					          }
				             }
		                       }
		
			
		}	
		

	
	],
	
	"outbounds": [
		{
			"protocol": "freedom",
			"settings": {}
		},
		{
			"protocol": "blackhole",
			"settings": {},
			"tag": "blocked"
		},
		{
			"protocol": "freedom",
			"settings": {},
			"tag": "direct"
		}

	],
	"dns": {
		"server": [
			"1.1.1.1",
			"1.0.0.1",
			"8.8.8.8",
			"8.8.4.4",
			"localhost"
		]
	},
	"routing": {
		"domainStrategy": "IPOnDemand",
		"rules": [
			{
				"type": "field",
				"ip": [
					"0.0.0.0/8",
					"10.0.0.0/8",
					"100.64.0.0/10",
					"127.0.0.0/8",
					"169.254.0.0/16",
					"172.16.0.0/12",
					"192.0.0.0/24",
					"192.0.2.0/24",
					"192.168.0.0/16",
					"198.18.0.0/15",
					"198.51.100.0/24",
					"203.0.113.0/24",
					"::1/128",
					"fc00::/7",
					"fe80::/10"
				],
				"outboundTag": "blocked"
			},
			{
				"type": "field",
				"inboundTag": ["tg-in"],
				"outboundTag": "tg-out"
			}
			,
			{
				"type": "field",
				"domain": [
				"domain:aaaa.com",
				"domain:bbbb.com" 
				],
				"outboundTag": "blocked"
			}
		]
	},
	"transport": {
		"kcpSettings": {
            "uplinkCapacity": 100,
            "downlinkCapacity": 100,
            "congestion": true
        },
		"sockopt": {
			"tcpFastOpen": true
		}
	}
}






```




```bash
trojan v2ray nginx共用配置 conf

server
    {
        listen 80;
	server_name 666.vvppmm.xyz;
        index index.html index.htm;
        root  /home/666.vvppmm.xyz;

        location / {
        proxy_pass https://www.asmag.com;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        }


	location /666api {
      if ($http_upgrade != "websocket") { 
          return 403;
      }
      proxy_redirect off;
      proxy_pass http://127.0.0.1:54123;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location ~ /.well-known {
            allow all;
    }

    location ~ .*\.(txt)$   
    {
        expires      30d;

    }
     
	 location ~ /\.
    {
            deny all;
     }

    }


caddy 共用需先申请好证书

:80 {
root /var/www/site
gzip
browse
proxy / http://majuredata.com
proxy /666api localhost:54123 {
     websocket
     header_upstream -Origin
     }
}

===========trojan 官方脚本=========


sudo bash -c "$(wget -O- https://raw.githubusercontent.com/trojan-gfw/trojan-quickstart/master/trojan-quickstart.sh)"


config文件所在

/usr/local/etc/trojan

vi /usr/local/etc/trojan/config.json

trojan重启      systemctl restart trojan.service
trojan状态      systemctl status trojan.service
trojan开机启动  systemctl enable trojan.service
trojan停止      systemctl stop trojan.service

```


trojan 安装


```bash
注：在cnetos7  上要改
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sed -i 's/SELINUX=permissive/SELINUX=disabled/g' /etc/selinux/config

注：可下载不同架构二进制包
wget https://github.com/trojan-gfw/trojan/releases/download/v1.13.0/trojan-1.13.0-linux-amd64.tar.xz

tar xf trojan-1.13.0-linux-amd64.tar.xz -C /etc/

vi /etc/trojan/config.json


cat > /lib/systemd/system/trojan.service <<-EOF
[Unit]  
Description=trojan  
After=network.target  
   
[Service]  
Type=simple  
PIDFile=/etc/trojan/trojan.pid
ExecStart=/etc/trojan/trojan -c "/etc/trojan/config.json"  
ExecReload=  
ExecStop=/etc/trojan/trojan  
PrivateTmp=true  
   
[Install]  
WantedBy=multi-user.target
EOF

chmod +x /usr/lib/systemd/system/trojan.service
chmod +x /lib/systemd/system/trojan.service
chown -R root:root /etc/trojan
systemctl start trojan.service
systemctl status trojan.service
systemctl enable trojan.service

```
