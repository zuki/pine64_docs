# Pine64+のデバイスツリー

```bash
/dts-v1/;

/ {
	interrupt-parent = <0x01>;
	#address-cells = <0x01>;
	#size-cells = <0x01>;
	model = "Pine64+";
	compatible = "pine64,pine64-plus", "allwinner,sun50i-a64";

	chosen {
		#address-cells = <0x01>;
		#size-cells = <0x01>;
		ranges;
		stdout-path = "serial0:115200n8";

		framebuffer-lcd {
			compatible = "allwinner,simple-framebuffer", "simple-framebuffer";
			allwinner,pipeline = "mixer0-lcd0";
			clocks = <0x02 0x64 0x03 0x06>;
			status = "disabled";
		};

		framebuffer-hdmi {
			compatible = "allwinner,simple-framebuffer", "simple-framebuffer";
			allwinner,pipeline = "mixer1-lcd1-hdmi";
			clocks = <0x03 0x07 0x02 0x65 0x02 0x6e>;
			status = "disabled";
			vcc-hdmi-supply = <0x04>;
		};
	};

	cpus {
		#address-cells = <0x01>;
		#size-cells = <0x00>;

		cpu@0 {
			compatible = "arm,cortex-a53";
			device_type = "cpu";
			reg = <0x00>;
			enable-method = "psci";
			next-level-cache = <0x05>;
			clocks = <0x02 0x15>;
			clock-names = "cpu";
			#cooling-cells = <0x02>;
			operating-points-v2 = <0x06>;
			cpu-supply = <0x07>;
			phandle = <0x0a>;
		};

		cpu@1 {
			compatible = "arm,cortex-a53";
			device_type = "cpu";
			reg = <0x01>;
			enable-method = "psci";
			next-level-cache = <0x05>;
			clocks = <0x02 0x15>;
			clock-names = "cpu";
			#cooling-cells = <0x02>;
			operating-points-v2 = <0x06>;
			cpu-supply = <0x07>;
			phandle = <0x0b>;
		};

		cpu@2 {
			compatible = "arm,cortex-a53";
			device_type = "cpu";
			reg = <0x02>;
			enable-method = "psci";
			next-level-cache = <0x05>;
			clocks = <0x02 0x15>;
			clock-names = "cpu";
			#cooling-cells = <0x02>;
			operating-points-v2 = <0x06>;
			cpu-supply = <0x07>;
			phandle = <0x0c>;
		};

		cpu@3 {
			compatible = "arm,cortex-a53";
			device_type = "cpu";
			reg = <0x03>;
			enable-method = "psci";
			next-level-cache = <0x05>;
			clocks = <0x02 0x15>;
			clock-names = "cpu";
			#cooling-cells = <0x02>;
			operating-points-v2 = <0x06>;
			cpu-supply = <0x07>;
			phandle = <0x0d>;
		};

		l2-cache {
			compatible = "cache";
			cache-level = <0x02>;
			phandle = <0x05>;
		};
	};

	display-engine {
		compatible = "allwinner,sun50i-a64-display-engine";
		allwinner,pipelines = <0x08 0x09>;
		status = "okay";
	};

	opp-table-gpu {
		compatible = "operating-points-v2";
		phandle = <0x3a>;

		opp-120000000 {
			opp-hz = <0x00 0x7270e00>;
		};

		opp-312000000 {
			opp-hz = <0x00 0x1298be00>;
		};

		opp-432000000 {
			opp-hz = <0x00 0x19bfcc00>;
		};
	};

	/* clock/fixed-clock.yaml */
	osc24M_clk {
		#clock-cells = <0x00>;
		compatible = "fixed-clock";
		clock-frequency = <0x16e3600>;
		clock-output-names = "osc24M";
		phandle = <0x25>;
	};

	osc32k_clk {
		#clock-cells = <0x00>;
		compatible = "fixed-clock";
		clock-frequency = <0x8000>;
		clock-output-names = "ext-osc32k";
		phandle = <0x43>;
	};

	/* Power Management Unit: arm/pmu.yaml */
	pmu {
		compatible = "arm,cortex-a53-pmu";
		interrupts = <0x00 0x74 0x04 0x00 0x75 0x04 0x00 0x76 0x04 0x00 0x77 0x04>;
		interrupt-affinity = <0x0a 0x0b 0x0c 0x0d>;
	};

	/* Power State Coordination Interface: arm/psci.yaml */
	psci {
		compatible = "arm,psci-0.2";
		method = "smc";
	};

	sound {
		#address-cells = <0x01>;
		#size-cells = <0x00>;
		compatible = "simple-audio-card";
		simple-audio-card,name = "sun50i-a64-audio";
		simple-audio-card,aux-devs = <0x0e>;
		simple-audio-card,routing = "Left DAC", "DACL", "Right DAC", "DACR", "Headphone Jack", "HP", "ADCL", "Left ADC", "ADCR", "Right ADC", "MIC2", "Microphone Jack";
		status = "okay";
		simple-audio-card,widgets = "Microphone", "Microphone Jack", "Headphone", "Headphone Jack";

		simple-audio-card,dai-link@0 {
			format = "i2s";
			frame-master = <0x0f>;
			bitclock-master = <0x0f>;
			mclk-fs = <0x80>;

			cpu {
				sound-dai = <0x10>;
				phandle = <0x0f>;
			};

			codec {
				sound-dai = <0x11 0x00>;
			};
		};
	};

	/* timer/arm,arch_timer.yaml */
	/*   per-cpu timer */
	timer {
		compatible = "arm,armv8-timer";
		allwinner,erratum-unknown1;
		arm,no-tick-in-suspend;
		interrupts = <0x01 0x0d 0xf04 0x01 0x0e 0xf04 0x01 0x0b 0xf04 0x01 0x0a 0xf04>;
	};

	thermal-zones {

		cpu0-thermal {
			polling-delay-passive = <0x00>;
			polling-delay = <0x00>;
			thermal-sensors = <0x12 0x00>;

			cooling-maps {

				map0 {
					trip = <0x13>;
					cooling-device = <0x0a 0xffffffff 0xffffffff 0x0b 0xffffffff 0xffffffff 0x0c 0xffffffff 0xffffffff 0x0d 0xffffffff 0xffffffff>;
				};

				map1 {
					trip = <0x14>;
					cooling-device = <0x0a 0xffffffff 0xffffffff 0x0b 0xffffffff 0xffffffff 0x0c 0xffffffff 0xffffffff 0x0d 0xffffffff 0xffffffff>;
				};
			};

			trips {

				cpu_alert0 {
					temperature = <0x124f8>;
					hysteresis = <0x7d0>;
					type = "passive";
					phandle = <0x13>;
				};

				cpu_alert1 {
					temperature = <0x15f90>;
					hysteresis = <0x7d0>;
					type = "hot";
					phandle = <0x14>;
				};

				cpu_crit {
					temperature = <0x1adb0>;
					hysteresis = <0x7d0>;
					type = "critical";
				};
			};
		};

		gpu0-thermal {
			polling-delay-passive = <0x00>;
			polling-delay = <0x00>;
			thermal-sensors = <0x12 0x01>;
		};

		gpu1-thermal {
			polling-delay-passive = <0x00>;
			polling-delay = <0x00>;
			thermal-sensors = <0x12 0x02>;
		};
	};

	soc {
		compatible = "simple-bus";
		#address-cells = <0x01>;
		#size-cells = <0x01>;
		ranges;

		bus@1000000 {
			compatible = "allwinner,sun50i-a64-de2";
			reg = <0x1000000 0x400000>;
			allwinner,sram = <0x15 0x01>;
			#address-cells = <0x01>;
			#size-cells = <0x01>;
			ranges = <0x00 0x1000000 0x400000>;

			clock@0 {
				compatible = "allwinner,sun50i-a64-de2-clk";
				reg = <0x00 0x10000>;
				clocks = <0x02 0x34 0x02 0x63>;
				clock-names = "bus", "mod";
				resets = <0x02 0x1e>;
				#clock-cells = <0x01>;
				#reset-cells = <0x01>;
				phandle = <0x03>;
			};

			rotate@20000 {
				compatible = "allwinner,sun50i-a64-de2-rotate", "allwinner,sun8i-a83t-de2-rotate";
				reg = <0x20000 0x10000>;
				interrupts = <0x00 0x60 0x04>;
				clocks = <0x03 0x09 0x03 0x0a>;
				clock-names = "bus", "mod";
				resets = <0x03 0x03>;
			};

			mixer@100000 {
				compatible = "allwinner,sun50i-a64-de2-mixer-0";
				reg = <0x100000 0x100000>;
				clocks = <0x03 0x00 0x03 0x06>;
				clock-names = "bus", "mod";
				resets = <0x03 0x00>;
				phandle = <0x08>;

				ports {
					#address-cells = <0x01>;
					#size-cells = <0x00>;

					port@1 {
						#address-cells = <0x01>;
						#size-cells = <0x00>;
						reg = <0x01>;

						endpoint@0 {
							reg = <0x00>;
							remote-endpoint = <0x16>;
							phandle = <0x1a>;
						};

						endpoint@1 {
							reg = <0x01>;
							remote-endpoint = <0x17>;
							phandle = <0x1d>;
						};
					};
				};
			};

			mixer@200000 {
				compatible = "allwinner,sun50i-a64-de2-mixer-1";
				reg = <0x200000 0x100000>;
				clocks = <0x03 0x01 0x03 0x07>;
				clock-names = "bus", "mod";
				resets = <0x03 0x01>;
				phandle = <0x09>;

				ports {
					#address-cells = <0x01>;
					#size-cells = <0x00>;

					port@1 {
						#address-cells = <0x01>;
						#size-cells = <0x00>;
						reg = <0x01>;

						endpoint@0 {
							reg = <0x00>;
							remote-endpoint = <0x18>;
							phandle = <0x1b>;
						};

						endpoint@1 {
							reg = <0x01>;
							remote-endpoint = <0x19>;
							phandle = <0x1e>;
						};
					};
				};
			};
		};

		syscon@1c00000 {
			compatible = "allwinner,sun50i-a64-system-control";
			reg = <0x1c00000 0x1000>;
			#address-cells = <0x01>;
			#size-cells = <0x01>;
			ranges;
			phandle = <0x36>;

			sram@18000 {
				compatible = "mmio-sram";
				reg = <0x18000 0x28000>;
				#address-cells = <0x01>;
				#size-cells = <0x01>;
				ranges = <0x00 0x18000 0x28000>;

				sram-section@0 {
					compatible = "allwinner,sun50i-a64-sram-c";
					reg = <0x00 0x28000>;
					phandle = <0x15>;
				};
			};

			sram@1d00000 {
				compatible = "mmio-sram";
				reg = <0x1d00000 0x40000>;
				#address-cells = <0x01>;
				#size-cells = <0x01>;
				ranges = <0x00 0x1d00000 0x40000>;

				sram-section@0 {
					compatible = "allwinner,sun50i-a64-sram-c1", "allwinner,sun4i-a10-sram-c1";
					reg = <0x00 0x40000>;
					phandle = <0x20>;
				};
			};
		};
		/* dma/allwinner,sun50i-a64-dma.yaml */
		dma-controller@1c02000 {
			compatible = "allwinner,sun50i-a64-dma";
			reg = <0x1c02000 0x1000>;
			interrupts = <0x00 0x32 0x04>;
			clocks = <0x02 0x1e>;
			dma-channels = <0x08>;
			dma-requests = <0x1b>;
			resets = <0x02 0x07>;
			#dma-cells = <0x01>;
			phandle = <0x28>;
		};

		lcd-controller@1c0c000 {
			compatible = "allwinner,sun50i-a64-tcon-lcd", "allwinner,sun8i-a83t-tcon-lcd";
			reg = <0x1c0c000 0x1000>;
			interrupts = <0x00 0x56 0x04>;
			clocks = <0x02 0x2f 0x02 0x64>;
			clock-names = "ahb", "tcon-ch0";
			clock-output-names = "tcon-pixel-clock";
			#clock-cells = <0x00>;
			resets = <0x02 0x18 0x02 0x23>;
			reset-names = "lcd", "lvds";

			ports {
				#address-cells = <0x01>;
				#size-cells = <0x00>;

				port@0 {
					#address-cells = <0x01>;
					#size-cells = <0x00>;
					reg = <0x00>;

					endpoint@0 {
						reg = <0x00>;
						remote-endpoint = <0x1a>;
						phandle = <0x16>;
					};

					endpoint@1 {
						reg = <0x01>;
						remote-endpoint = <0x1b>;
						phandle = <0x18>;
					};
				};

				port@1 {
					#address-cells = <0x01>;
					#size-cells = <0x00>;
					reg = <0x01>;

					endpoint@1 {
						reg = <0x01>;
						remote-endpoint = <0x1c>;
						allwinner,tcon-channel = <0x01>;
						phandle = <0x3e>;
					};
				};
			};
		};

		lcd-controller@1c0d000 {
			compatible = "allwinner,sun50i-a64-tcon-tv", "allwinner,sun8i-a83t-tcon-tv";
			reg = <0x1c0d000 0x1000>;
			interrupts = <0x00 0x57 0x04>;
			clocks = <0x02 0x30 0x02 0x65>;
			clock-names = "ahb", "tcon-ch1";
			resets = <0x02 0x19>;
			reset-names = "lcd";

			ports {
				#address-cells = <0x01>;
				#size-cells = <0x00>;

				port@0 {
					#address-cells = <0x01>;
					#size-cells = <0x00>;
					reg = <0x00>;

					endpoint@0 {
						reg = <0x00>;
						remote-endpoint = <0x1d>;
						phandle = <0x17>;
					};

					endpoint@1 {
						reg = <0x01>;
						remote-endpoint = <0x1e>;
						phandle = <0x19>;
					};
				};

				port@1 {
					#address-cells = <0x01>;
					#size-cells = <0x00>;
					reg = <0x01>;

					endpoint@1 {
						reg = <0x01>;
						remote-endpoint = <0x1f>;
						phandle = <0x41>;
					};
				};
			};
		};

		video-codec@1c0e000 {
			compatible = "allwinner,sun50i-a64-video-engine";
			reg = <0x1c0e000 0x1000>;
			clocks = <0x02 0x2e 0x02 0x6a 0x02 0x5f>;
			clock-names = "ahb", "mod", "ram";
			resets = <0x02 0x17>;
			interrupts = <0x00 0x3a 0x04>;
			allwinner,sram = <0x20 0x01>;
		};

		/* SMHC0 : mmc/allwinner,sun4i-a10-mmc.yaml,
		 *         mmc/mmc-controller.yaml
		 */
		mmc@1c0f000 {
			compatible = "allwinner,sun50i-a64-mmc";
			reg = <0x1c0f000 0x1000>;
			/* bus, module, output, sample */
			clocks = <0x02 0x1f 0x02 0x4b>;
			clock-names = "ahb", "mmc";
			resets = <0x02 0x08>;
			reset-names = "ahb";
			interrupts = <0x00 0x3c 0x04>;		// 92
			max-frequency = <0x8f0d180>;		// 150 MHz
			status = "okay";
			#address-cells = <0x01>;
			#size-cells = <0x00>;
			pinctrl-names = "default";
			pinctrl-0 = <0x21>;
			/* Supply for the card power : phandle=0x22 : 3.3v */
			vmmc-supply = <0x22>;
			/* card detection will be done using the GPIO provided */
			cd-gpios = <0x23 0x05 0x06 0x01>;
			/* no physical write-protect line is present */
			disable-wp;
			/* Number of data lines */
			bus-width = <0x04>;
		};

		/* SMHC 1 */
		mmc@1c10000 {
			compatible = "allwinner,sun50i-a64-mmc";
			reg = <0x1c10000 0x1000>;
			clocks = <0x02 0x20 0x02 0x4c>;
			clock-names = "ahb", "mmc";
			resets = <0x02 0x09>;
			reset-names = "ahb";
			interrupts = <0x00 0x3d 0x04>;
			max-frequency = <0x8f0d180>;
			status = "disabled";
			#address-cells = <0x01>;
			#size-cells = <0x00>;
		};

		/* SMHC 2 */
		mmc@1c11000 {
			compatible = "allwinner,sun50i-a64-emmc";
			reg = <0x1c11000 0x1000>;
			clocks = <0x02 0x21 0x02 0x4d>;
			clock-names = "ahb", "mmc";
			resets = <0x02 0x0a>;
			reset-names = "ahb";
			interrupts = <0x00 0x3e 0x04>;
			max-frequency = <0x8f0d180>;
			status = "disabled";
			#address-cells = <0x01>;
			#size-cells = <0x00>;
		};

		eeprom@1c14000 {
			compatible = "allwinner,sun50i-a64-sid";
			reg = <0x1c14000 0x400>;
			#address-cells = <0x01>;
			#size-cells = <0x01>;

			thermal-sensor-calibration@34 {
				reg = <0x34 0x08>;
				phandle = <0x2a>;
			};
		};

		crypto@1c15000 {
			compatible = "allwinner,sun50i-a64-crypto";
			reg = <0x1c15000 0x1000>;
			interrupts = <0x00 0x5e 0x04>;
			clocks = <0x02 0x1d 0x02 0x4f>;
			clock-names = "bus", "mod";
			resets = <0x02 0x06>;
		};

		/* mailbox/allwinner,sun6i-a31-msgbox.yaml */
		mailbox@1c17000 {
			compatible = "allwinner,sun50i-a64-msgbox", "allwinner,sun6i-a31-msgbox";
			reg = <0x1c17000 0x1000>;
			clocks = <0x02 0x36>;
			resets = <0x02 0x20>;
			interrupts = <0x00 0x31 0x04>;
			#mbox-cells = <0x01>;
		};

		/* USB-OTG Device : usb/allwinner,sun4i-a10-musb.yaml */
		usb@1c19000 {
			compatible = "allwinner,sun8i-a33-musb";
			reg = <0x1c19000 0x400>;
			clocks = <0x02 0x29>;
			resets = <0x02 0x12>;
			interrupts = <0x00 0x47 0x04>;
			interrupt-names = "mc";
			phys = <0x24 0x00>;			// phy@1c19400
			phy-names = "usb";
			extcon = <0x24 0x00>;		// phy@1c19400
			dr_mode = "host";			// Hostとして使用
			status = "okay";
		};

		/* Allwinner A64 USB PHY :
		 *         phy/allwinner,sun50i-a64-usb-phy.yaml */
		phy@1c19400 {
			compatible = "allwinner,sun50i-a64-usb-phy";
			reg = <0x1c19400 0x14 0x1c1a800 0x04 0x1c1b800 0x04>;
			reg-names = "phy_ctrl", "pmu0", "pmu1";
			clocks = <0x02 0x56 0x02 0x57>;
			clock-names = "usb0_phy", "usb1_phy";
			resets = <0x02 0x00 0x02 0x01>;
			reset-names = "usb0_reset", "usb1_reset";
			status = "okay";
			#phy-cells = <0x01>;
			phandle = <0x24>;
		};

		/* USB EHCI Controller 上端 : usb/generic-ehci.yaml */
		usb@1c1a000 {
			compatible = "allwinner,sun50i-a64-ehci", "generic-ehci";
			reg = <0x1c1a000 0x100>;
			interrupts = <0x00 0x48 0x04>;
			clocks = <0x02 0x2c 0x02 0x2a 0x02 0x5b>;
			resets = <0x02 0x15 0x02 0x13>;
			phys = <0x24 0x00>;
			phy-names = "usb";
			status = "okay";
		};

		/* USB OHCI Controller 上端 : usb/generic-ohci.yaml */
		usb@1c1a400 {
			compatible = "allwinner,sun50i-a64-ohci", "generic-ohci";
			reg = <0x1c1a400 0x100>;
			interrupts = <0x00 0x49 0x04>;
			clocks = <0x02 0x2c 0x02 0x5b>;
			resets = <0x02 0x15>;
			phys = <0x24 0x00>;
			phy-names = "usb";
			status = "okay";
		};

		/* USB EHCI Controller 下端 : usb/generic-ehci.yaml  */
		usb@1c1b000 {
			compatible = "allwinner,sun50i-a64-ehci", "generic-ehci";
			reg = <0x1c1b000 0x100>;
			interrupts = <0x00 0x4a 0x04>;
			clocks = <0x02 0x2d 0x02 0x2b 0x02 0x5d>;
			resets = <0x02 0x16 0x02 0x14>;
			phys = <0x24 0x01>;
			phy-names = "usb";
			status = "okay";
		};

		/* USB OHCI Controller 下端 : usb/generic-ohci.yaml */
		usb@1c1b400 {
			compatible = "allwinner,sun50i-a64-ohci", "generic-ohci";
			reg = <0x1c1b400 0x100>;
			interrupts = <0x00 0x4b 0x04>;
			clocks = <0x02 0x2d 0x02 0x5d>;
			resets = <0x02 0x16>;
			phys = <0x24 0x01>;
			phy-names = "usb";
			status = "okay";
		};

		/* Allwinner Clock Control Unit :
		 * 		clock/allwinner,sun4i-a10-ccu.yaml */
		clock@1c20000 {
			compatible = "allwinner,sun50i-a64-ccu";
			reg = <0x1c20000 0x400>;
			/* 24MHz, 32kHz, Internal Oscillator */
			clocks = <0x25 0x26 0x00>;
			clock-names = "hosc", "losc";
			#clock-cells = <0x01>;
			#reset-cells = <0x01>;
			phandle = <0x02>;
		};

		/* Allwinner Pin Controller :
		 * 		pinctrl/allwinner,sun4i-a10-pinctrl.yaml */
		pinctrl@1c20800 {
			compatible = "allwinner,sun50i-a64-pinctrl";
			reg = <0x1c20800 0x400>;
			interrupt-parent = <0x27>;
			/* 43 (PB_EINT), 49 (PG_EINT), 53 (PH_EINT) */
			interrupts = <0x00 0x0b 0x04 0x00 0x11 0x04 0x00 0x15 0x04>;
			clocks = <0x02 0x3a 0x25 0x26 0x00>;
			clock-names = "apb", "hosc", "losc";
			gpio-controller;
			#gpio-cells = <0x03>;
			interrupt-controller;
			#interrupt-cells = <0x03>;
			phandle = <0x23>;

			csi-pins {
				pins = "PE0", "PE2", "PE3", "PE4", "PE5", "PE6", "PE7", "PE8", "PE9", "PE10", "PE11";
				function = "csi";
				phandle = <0x3c>;
			};

			i2c0-pins {
				pins = "PH0", "PH1";
				function = "i2c0";
				phandle = <0x31>;
			};

			i2c1-pins {
				pins = "PH2", "PH3";
				function = "i2c1";
				bias-pull-up;
				phandle = <0x32>;
			};

			i2c2-pins {
				pins = "PE14", "PE15";
				function = "i2c2";
				phandle = <0x33>;
			};

			mmc0-pins {
				pins = "PF0", "PF1", "PF2", "PF3", "PF4", "PF5";
				function = "mmc0";
				drive-strength = <0x1e>;
				bias-pull-up;
				phandle = <0x21>;
			};

			mmc1-pins {
				pins = "PG0", "PG1", "PG2", "PG3", "PG4", "PG5";
				function = "mmc1";
				drive-strength = <0x1e>;
				bias-pull-up;
			};

			mmc2-pins {
				pins = "PC5", "PC6", "PC8", "PC9", "PC10", "PC11", "PC12", "PC13", "PC14", "PC15", "PC16";
				function = "mmc2";
				drive-strength = <0x1e>;
				bias-pull-up;
			};

			mmc2-ds-pin {
				pins = "PC1";
				function = "mmc2";
				drive-strength = <0x1e>;
				bias-pull-up;
			};

			pwm-pin {
				pins = "PD22";
				function = "pwm";
				phandle = <0x3b>;
			};

			rmii-pins {
				pins = "PD10", "PD11", "PD13", "PD14", "PD17", "PD18", "PD19", "PD20", "PD22", "PD23";
				function = "emac";
				drive-strength = <0x28>;
			};

			rgmii-pins {
				pins = "PD8", "PD9", "PD10", "PD11", "PD12", "PD13", "PD15", "PD16", "PD17", "PD18", "PD19", "PD20", "PD21", "PD22", "PD23";
				function = "emac";
				drive-strength = <0x28>;
				phandle = <0x37>;
			};

			spdif-tx-pin {
				pins = "PH8";
				function = "spdif";
				phandle = <0x29>;
			};

			spi0-pins {
				pins = "PC0", "PC1", "PC2", "PC3";
				function = "spi0";
				phandle = <0x34>;
			};

			spi1-pins {
				pins = "PD0", "PD1", "PD2", "PD3";
				function = "spi1";
				phandle = <0x35>;
			};

			uart0-pb-pins {
				pins = "PB8", "PB9";
				function = "uart0";
				phandle = <0x2b>;
			};

			uart1-pins {
				pins = "PG6", "PG7";
				function = "uart1";
				phandle = <0x2c>;
			};

			uart1-rts-cts-pins {
				pins = "PG8", "PG9";
				function = "uart1";
				phandle = <0x2d>;
			};

			uart2-pins {
				pins = "PB0", "PB1";
				function = "uart2";
				phandle = <0x2e>;
			};

			uart3-pins {
				pins = "PD0", "PD1";
				function = "uart3";
				phandle = <0x2f>;
			};

			uart4-pins {
				pins = "PD2", "PD3";
				function = "uart4";
				phandle = <0x30>;
			};

			uart4-rts-cts-pins {
				pins = "PD4", "PD5";
				function = "uart4";
			};
		};

		/* Allwinner Timer : timer/allwinner,sun4i-a10-timer.yaml */
		timer@1c20c00 {
			compatible = "allwinner,sun50i-a64-timer", "allwinner,sun8i-a23-timer";
			reg = <0x1c20c00 0xa0>;
			interrupts = <0x00 0x12 0x04 0x00 0x13 0x04>;
			clocks = <0x25>;		// 25MHz
		};

		watchdog@1c20ca0 {
			compatible = "allwinner,sun50i-a64-wdt", "allwinner,sun6i-a31-wdt";
			reg = <0x1c20ca0 0x20>;
			interrupts = <0x00 0x19 0x04>;
			clocks = <0x25>;
		};

		spdif@1c21000 {
			#sound-dai-cells = <0x00>;
			compatible = "allwinner,sun50i-a64-spdif", "allwinner,sun8i-h3-spdif";
			reg = <0x1c21000 0x400>;
			interrupts = <0x00 0x0c 0x04>;
			clocks = <0x02 0x39 0x02 0x55>;
			resets = <0x02 0x25>;
			clock-names = "apb", "spdif";
			dmas = <0x28 0x02>;
			dma-names = "tx";
			pinctrl-names = "default";
			pinctrl-0 = <0x29>;
			status = "disabled";
		};

		lradc@1c21800 {
			compatible = "allwinner,sun50i-a64-lradc", "allwinner,sun8i-a83t-r-lradc";
			reg = <0x1c21800 0x400>;
			interrupt-parent = <0x27>;
			interrupts = <0x00 0x1e 0x04>;
			status = "disabled";
		};

		i2s@1c22000 {
			#sound-dai-cells = <0x00>;
			compatible = "allwinner,sun50i-a64-i2s", "allwinner,sun8i-h3-i2s";
			reg = <0x1c22000 0x400>;
			interrupts = <0x00 0x0d 0x04>;
			clocks = <0x02 0x3c 0x02 0x52>;
			clock-names = "apb", "mod";
			resets = <0x02 0x27>;
			dma-names = "rx", "tx";
			dmas = <0x28 0x03 0x28 0x03>;
			status = "disabled";
		};

		i2s@1c22400 {
			#sound-dai-cells = <0x00>;
			compatible = "allwinner,sun50i-a64-i2s", "allwinner,sun8i-h3-i2s";
			reg = <0x1c22400 0x400>;
			interrupts = <0x00 0x0e 0x04>;
			clocks = <0x02 0x3d 0x02 0x53>;
			clock-names = "apb", "mod";
			resets = <0x02 0x28>;
			dma-names = "rx", "tx";
			dmas = <0x28 0x04 0x28 0x04>;
			status = "disabled";
		};

		i2s@1c22800 {
			#sound-dai-cells = <0x00>;
			compatible = "allwinner,sun50i-a64-i2s", "allwinner,sun8i-h3-i2s";
			reg = <0x1c22800 0x400>;
			interrupts = <0x00 0x0f 0x04>;
			clocks = <0x02 0x3e 0x02 0x54>;
			clock-names = "apb", "mod";
			resets = <0x02 0x29>;
			dma-names = "rx", "tx";
			dmas = <0x28 0x1b 0x28 0x1b>;
			status = "disabled";
		};

		dai@1c22c00 {
			#sound-dai-cells = <0x00>;
			compatible = "allwinner,sun50i-a64-codec-i2s";
			reg = <0x1c22c00 0x200>;
			interrupts = <0x00 0x1d 0x04>;
			clocks = <0x02 0x38 0x02 0x6b>;
			clock-names = "apb", "mod";
			resets = <0x02 0x24>;
			dmas = <0x28 0x0f 0x28 0x0f>;
			dma-names = "rx", "tx";
			status = "okay";
			phandle = <0x10>;
		};

		codec@1c22e00 {
			#sound-dai-cells = <0x01>;
			compatible = "allwinner,sun50i-a64-codec", "allwinner,sun8i-a33-codec";
			reg = <0x1c22e00 0x600>;
			interrupts = <0x00 0x1c 0x04>;
			clocks = <0x02 0x38 0x02 0x6b>;
			clock-names = "bus", "mod";
			status = "okay";
			phandle = <0x11>;
		};

		thermal-sensor@1c25000 {
			compatible = "allwinner,sun50i-a64-ths";
			reg = <0x1c25000 0x100>;
			clocks = <0x02 0x3b 0x02 0x49>;
			clock-names = "bus", "mod";
			interrupts = <0x00 0x1f 0x04>;
			resets = <0x02 0x26>;
			nvmem-cells = <0x2a>;
			nvmem-cell-names = "calibration";
			#thermal-sensor-cells = <0x01>;
			phandle = <0x12>;
		};

		/* Synopsys DesignWare ABP UART : UART0
		 * 		serial/snps-dw-apb-uart.yaml, serial.yaml */
		serial@1c28000 {
			compatible = "snps,dw-apb-uart";
			reg = <0x1c28000 0x400>;
			interrupts = <0x00 0x00 0x04>;
			reg-shift = <0x02>;
			reg-io-width = <0x04>;
			clocks = <0x02 0x43>;
			resets = <0x02 0x2e>;
			status = "okay";
			pinctrl-names = "default";
			pinctrl-0 = <0x2b>;
		};

		serial@1c28400 {
			compatible = "snps,dw-apb-uart";
			reg = <0x1c28400 0x400>;
			interrupts = <0x00 0x01 0x04>;
			reg-shift = <0x02>;
			reg-io-width = <0x04>;
			clocks = <0x02 0x44>;
			resets = <0x02 0x2f>;
			status = "disabled";
			pinctrl-names = "default";
			pinctrl-0 = <0x2c 0x2d>;
		};

		serial@1c28800 {
			compatible = "snps,dw-apb-uart";
			reg = <0x1c28800 0x400>;
			interrupts = <0x00 0x02 0x04>;
			reg-shift = <0x02>;
			reg-io-width = <0x04>;
			clocks = <0x02 0x45>;
			resets = <0x02 0x30>;
			status = "disabled";
			pinctrl-names = "default";
			pinctrl-0 = <0x2e>;
		};

		serial@1c28c00 {
			compatible = "snps,dw-apb-uart";
			reg = <0x1c28c00 0x400>;
			interrupts = <0x00 0x03 0x04>;
			reg-shift = <0x02>;
			reg-io-width = <0x04>;
			clocks = <0x02 0x46>;
			resets = <0x02 0x31>;
			status = "disabled";
			pinctrl-names = "default";
			pinctrl-0 = <0x2f>;
		};

		serial@1c29000 {
			compatible = "snps,dw-apb-uart";
			reg = <0x1c29000 0x400>;
			interrupts = <0x00 0x04 0x04>;
			reg-shift = <0x02>;
			reg-io-width = <0x04>;
			clocks = <0x02 0x47>;
			resets = <0x02 0x32>;
			status = "disabled";
			pinctrl-names = "default";
			pinctrl-0 = <0x30>;
		};

		i2c@1c2ac00 {
			compatible = "allwinner,sun6i-a31-i2c";
			reg = <0x1c2ac00 0x400>;
			interrupts = <0x00 0x06 0x04>;
			clocks = <0x02 0x3f>;
			resets = <0x02 0x2a>;
			pinctrl-names = "default";
			pinctrl-0 = <0x31>;
			status = "disabled";
			#address-cells = <0x01>;
			#size-cells = <0x00>;
		};

		i2c@1c2b000 {
			compatible = "allwinner,sun6i-a31-i2c";
			reg = <0x1c2b000 0x400>;
			interrupts = <0x00 0x07 0x04>;
			clocks = <0x02 0x40>;
			resets = <0x02 0x2b>;
			pinctrl-names = "default";
			pinctrl-0 = <0x32>;
			status = "okay";
			#address-cells = <0x01>;
			#size-cells = <0x00>;
		};

		i2c@1c2b400 {
			compatible = "allwinner,sun6i-a31-i2c";
			reg = <0x1c2b400 0x400>;
			interrupts = <0x00 0x08 0x04>;
			clocks = <0x02 0x41>;
			resets = <0x02 0x2c>;
			pinctrl-names = "default";
			pinctrl-0 = <0x33>;
			status = "disabled";
			#address-cells = <0x01>;
			#size-cells = <0x00>;
		};

		spi@1c68000 {
			compatible = "allwinner,sun8i-h3-spi";
			reg = <0x1c68000 0x1000>;
			interrupts = <0x00 0x41 0x04>;
			clocks = <0x02 0x27 0x02 0x50>;
			clock-names = "ahb", "mod";
			dmas = <0x28 0x17 0x28 0x17>;
			dma-names = "rx", "tx";
			pinctrl-names = "default";
			pinctrl-0 = <0x34>;
			resets = <0x02 0x10>;
			status = "disabled";
			num-cs = <0x01>;
			#address-cells = <0x01>;
			#size-cells = <0x00>;
		};

		spi@1c69000 {
			compatible = "allwinner,sun8i-h3-spi";
			reg = <0x1c69000 0x1000>;
			interrupts = <0x00 0x42 0x04>;
			clocks = <0x02 0x28 0x02 0x51>;
			clock-names = "ahb", "mod";
			dmas = <0x28 0x18 0x28 0x18>;
			dma-names = "rx", "tx";
			pinctrl-names = "default";
			pinctrl-0 = <0x35>;
			resets = <0x02 0x11>;
			status = "disabled";
			num-cs = <0x01>;
			#address-cells = <0x01>;
			#size-cells = <0x00>;
		};

		/* Allwinner EMAC : net/allwinner,sun8i-a83t-emac.yaml,
		 * 		net/snps,dwmac.yaml */
		ethernet@1c30000 {
			compatible = "allwinner,sun50i-a64-emac";
			syscon = <0x36>;
			reg = <0x1c30000 0x10000>;
			interrupts = <0x00 0x52 0x04>;
			interrupt-names = "macirq";
			resets = <0x02 0x0d>;
			reset-names = "stmmaceth";
			clocks = <0x02 0x24>;
			clock-names = "stmmaceth";
			status = "okay";
			pinctrl-names = "default";
			pinctrl-0 = <0x37>;
			phy-mode = "rgmii-txid";
			phy-handle = <0x38>;
			phy-supply = <0x39>;

			mdio {
				compatible = "snps,dwmac-mdio";
				#address-cells = <0x01>;
				#size-cells = <0x00>;

				ethernet-phy@1 {
					compatible = "ethernet-phy-ieee802.3-c22";
					reg = <0x01>;
					phandle = <0x38>;
				};
			};
		};

		gpu@1c40000 {
			compatible = "allwinner,sun50i-a64-mali", "arm,mali-400";
			reg = <0x1c40000 0x10000>;
			interrupts = <0x00 0x61 0x04 0x00 0x62 0x04 0x00 0x63 0x04 0x00 0x64 0x04 0x00 0x66 0x04 0x00 0x67 0x04 0x00 0x65 0x04>;
			interrupt-names = "gp", "gpmmu", "pp0", "ppmmu0", "pp1", "ppmmu1", "pmu";
			clocks = <0x02 0x35 0x02 0x72>;
			clock-names = "bus", "core";
			resets = <0x02 0x1f>;
			operating-points-v2 = <0x3a>;
		};

		/* ARM GIC v1/2 : interrupt-controller/arm,gic.yaml */
		interrupt-controller@1c81000 {
			compatible = "arm,gic-400";
			/* GICD, GICC, GICH, GICV */
			reg = <0x1c81000 0x1000 0x1c82000 0x2000 0x1c84000 0x2000 0x1c86000 0x2000>;
			/* VGICのメンテナンス割り込み: PPI 9, ここでは無視できる */
			interrupts = <0x01 0x09 0xf04>;
			interrupt-controller;
			#interrupt-cells = <0x03>;
			phandle = <0x01>;
		};

		pwm@1c21400 {
			compatible = "allwinner,sun50i-a64-pwm", "allwinner,sun5i-a13-pwm";
			reg = <0x1c21400 0x400>;
			clocks = <0x25>;
			pinctrl-names = "default";
			pinctrl-0 = <0x3b>;
			#pwm-cells = <0x03>;
			status = "disabled";
		};

		dram-controller@1c62000 {
			compatible = "allwinner,sun50i-a64-mbus";
			reg = <0x1c62000 0x1000 0x1c63000 0x1000>;
			reg-names = "mbus", "dram";
			clocks = <0x02 0x70 0x02 0x5e 0x02 0x23>;
			clock-names = "mbus", "dram", "bus";
			interrupts = <0x00 0x45 0x04>;
			#address-cells = <0x01>;
			#size-cells = <0x01>;
			dma-ranges = <0x00 0x40000000 0xc0000000>;
			#interconnect-cells = <0x01>;
			phandle = <0x3f>;
		};

		csi@1cb0000 {
			compatible = "allwinner,sun50i-a64-csi";
			reg = <0x1cb0000 0x1000>;
			interrupts = <0x00 0x54 0x04>;
			clocks = <0x02 0x32 0x02 0x68 0x02 0x60>;
			clock-names = "bus", "mod", "ram";
			resets = <0x02 0x1b>;
			pinctrl-names = "default";
			pinctrl-0 = <0x3c>;
			status = "disabled";
		};

		dsi@1ca0000 {
			compatible = "allwinner,sun50i-a64-mipi-dsi";
			reg = <0x1ca0000 0x1000>;
			interrupts = <0x00 0x59 0x04>;
			clocks = <0x02 0x1c>;
			resets = <0x02 0x05>;
			phys = <0x3d>;
			phy-names = "dphy";
			status = "disabled";
			#address-cells = <0x01>;
			#size-cells = <0x00>;

			port {

				endpoint {
					remote-endpoint = <0x3e>;
					phandle = <0x1c>;
				};
			};
		};

		d-phy@1ca1000 {
			compatible = "allwinner,sun50i-a64-mipi-dphy", "allwinner,sun6i-a31-mipi-dphy";
			reg = <0x1ca1000 0x1000>;
			clocks = <0x02 0x1c 0x02 0x71>;
			clock-names = "bus", "mod";
			resets = <0x02 0x05>;
			status = "disabled";
			#phy-cells = <0x00>;
			phandle = <0x3d>;
		};

		deinterlace@1e00000 {
			compatible = "allwinner,sun50i-a64-deinterlace", "allwinner,sun8i-h3-deinterlace";
			reg = <0x1e00000 0x20000>;
			clocks = <0x02 0x31 0x02 0x66 0x02 0x61>;
			clock-names = "bus", "mod", "ram";
			resets = <0x02 0x1a>;
			interrupts = <0x00 0x5d 0x04>;
			interconnects = <0x3f 0x09>;
			interconnect-names = "dma-mem";
		};

		hdmi@1ee0000 {
			compatible = "allwinner,sun50i-a64-dw-hdmi", "allwinner,sun8i-a83t-dw-hdmi";
			reg = <0x1ee0000 0x10000>;
			reg-io-width = <0x01>;
			interrupts = <0x00 0x58 0x04>;
			clocks = <0x02 0x33 0x02 0x6f 0x02 0x6e 0x26 0x00>;
			clock-names = "iahb", "isfr", "tmds", "cec";
			resets = <0x02 0x1d>;
			reset-names = "ctrl";
			phys = <0x40>;
			phy-names = "phy";
			status = "okay";
			hvcc-supply = <0x04>;

			ports {
				#address-cells = <0x01>;
				#size-cells = <0x00>;

				port@0 {
					reg = <0x00>;

					endpoint {
						remote-endpoint = <0x41>;
						phandle = <0x1f>;
					};
				};

				port@1 {
					reg = <0x01>;

					endpoint {
						remote-endpoint = <0x42>;
						phandle = <0x49>;
					};
				};
			};
		};

		hdmi-phy@1ef0000 {
			compatible = "allwinner,sun50i-a64-hdmi-phy";
			reg = <0x1ef0000 0x10000>;
			clocks = <0x02 0x33 0x02 0x6f 0x02 0x07>;
			clock-names = "bus", "mod", "pll-0";
			resets = <0x02 0x1c>;
			reset-names = "phy";
			#phy-cells = <0x00>;
			phandle = <0x40>;
		};

		rtc@1f00000 {
			compatible = "allwinner,sun50i-a64-rtc", "allwinner,sun8i-h3-rtc";
			reg = <0x1f00000 0x400>;
			interrupt-parent = <0x27>;
			interrupts = <0x00 0x28 0x04 0x00 0x29 0x04>;
			clock-output-names = "osc32k", "osc32k-out", "iosc";
			clocks = <0x43>;
			#clock-cells = <0x01>;
			phandle = <0x26>;
		};

		interrupt-controller@1f00c00 {
			compatible = "allwinner,sun50i-a64-r-intc", "allwinner,sun6i-a31-r-intc";
			interrupt-controller;
			#interrupt-cells = <0x03>;
			reg = <0x1f00c00 0x400>;
			interrupts = <0x00 0x20 0x04>;
			phandle = <0x27>;
		};

		clock@1f01400 {
			compatible = "allwinner,sun50i-a64-r-ccu";
			reg = <0x1f01400 0x100>;
			clocks = <0x25 0x26 0x00 0x26 0x02 0x02 0x0b>;
			clock-names = "hosc", "losc", "iosc", "pll-periph";
			#clock-cells = <0x01>;
			#reset-cells = <0x01>;
			phandle = <0x45>;
		};

		codec-analog@1f015c0 {
			compatible = "allwinner,sun50i-a64-codec-analog";
			reg = <0x1f015c0 0x04>;
			status = "okay";
			cpvdd-supply = <0x44>;
			phandle = <0x0e>;
		};

		i2c@1f02400 {
			compatible = "allwinner,sun50i-a64-i2c", "allwinner,sun6i-a31-i2c";
			reg = <0x1f02400 0x400>;
			interrupts = <0x00 0x2c 0x04>;
			clocks = <0x45 0x09>;
			resets = <0x45 0x05>;
			status = "disabled";
			#address-cells = <0x01>;
			#size-cells = <0x00>;
		};

		ir@1f02000 {
			compatible = "allwinner,sun50i-a64-ir", "allwinner,sun6i-a31-ir";
			reg = <0x1f02000 0x400>;
			clocks = <0x45 0x04 0x45 0x0b>;
			clock-names = "apb", "ir";
			resets = <0x45 0x00>;
			interrupts = <0x00 0x25 0x04>;
			pinctrl-names = "default";
			pinctrl-0 = <0x46>;
			status = "disabled";
		};

		pwm@1f03800 {
			compatible = "allwinner,sun50i-a64-pwm", "allwinner,sun5i-a13-pwm";
			reg = <0x1f03800 0x400>;
			clocks = <0x25>;
			pinctrl-names = "default";
			pinctrl-0 = <0x47>;
			#pwm-cells = <0x03>;
			status = "disabled";
		};

		pinctrl@1f02c00 {
			compatible = "allwinner,sun50i-a64-r-pinctrl";
			reg = <0x1f02c00 0x400>;
			interrupt-parent = <0x27>;
			interrupts = <0x00 0x2d 0x04>;
			clocks = <0x45 0x03 0x25 0x43>;
			clock-names = "apb", "hosc", "losc";
			gpio-controller;
			#gpio-cells = <0x03>;
			interrupt-controller;
			#interrupt-cells = <0x03>;

			r-i2c-pl89-pins {
				pins = "PL8", "PL9";
				function = "s_i2c";
			};

			r-ir-rx-pin {
				pins = "PL11";
				function = "s_cir_rx";
				phandle = <0x46>;
			};

			r-pwm-pin {
				pins = "PL10";
				function = "s_pwm";
				phandle = <0x47>;
			};

			r-rsb-pins {
				pins = "PL0", "PL1";
				function = "s_rsb";
				phandle = <0x48>;
			};
		};

		rsb@1f03400 {
			compatible = "allwinner,sun8i-a23-rsb";
			reg = <0x1f03400 0x400>;
			interrupts = <0x00 0x27 0x04>;
			clocks = <0x45 0x06>;
			clock-frequency = <0x2dc6c0>;
			resets = <0x45 0x02>;
			pinctrl-names = "default";
			pinctrl-0 = <0x48>;
			status = "okay";
			#address-cells = <0x01>;
			#size-cells = <0x00>;

			pmic@3a3 {
				compatible = "x-powers,axp803";
				reg = <0x3a3>;
				interrupt-parent = <0x27>;
				interrupts = <0x00 0x20 0x08>;
				interrupt-controller;
				#interrupt-cells = <0x01>;

				ac-power {
					compatible = "x-powers,axp803-ac-power-supply", "x-powers,axp813-ac-power-supply";
					status = "okay";
				};

				adc {
					compatible = "x-powers,axp803-adc", "x-powers,axp813-adc";
					#io-channel-cells = <0x01>;
				};

				gpio {
					compatible = "x-powers,axp803-gpio", "x-powers,axp813-gpio";
					gpio-controller;
					#gpio-cells = <0x02>;

					gpio0-ldo-pin {
						pins = "GPIO0";
						function = "ldo";
					};

					gpio1-ldo-pin {
						pins = "GPIO1";
						function = "ldo";
					};
				};

				battery-power {
					compatible = "x-powers,axp803-battery-power-supply", "x-powers,axp813-battery-power-supply";
					status = "okay";
				};

				regulators {
					x-powers,dcdc-freq = <0xbb8>;

					aldo1 {
						regulator-name = "aldo1";
					};

					aldo2 {
						regulator-name = "vcc-pl";
						regulator-always-on;
						regulator-min-microvolt = <0x1b7740>;
						regulator-max-microvolt = <0x325aa0>;
					};

					aldo3 {
						regulator-name = "vcc-pll-avcc";
						regulator-always-on;
						regulator-min-microvolt = <0x2dc6c0>;
						regulator-max-microvolt = <0x2dc6c0>;
					};

					dc1sw {
						regulator-name = "vcc-phy";
						regulator-enable-ramp-delay = <0x186a0>;
						phandle = <0x39>;
					};

					dcdc1 {
						regulator-name = "vcc-3v3";
						regulator-always-on;
						regulator-min-microvolt = <0x325aa0>;
						regulator-max-microvolt = <0x325aa0>;
						phandle = <0x22>;
					};

					dcdc2 {
						regulator-name = "vdd-cpux";
						regulator-always-on;
						regulator-min-microvolt = <0xfde80>;
						regulator-max-microvolt = <0x13d620>;
						phandle = <0x07>;
					};

					dcdc3 {
						regulator-name = "dcdc3";
					};

					dcdc4 {
						regulator-name = "dcdc4";
					};

					dcdc5 {
						regulator-name = "vcc-dram";
						regulator-always-on;
						regulator-min-microvolt = <0x14c080>;
						regulator-max-microvolt = <0x14c080>;
					};

					dcdc6 {
						regulator-name = "vdd-sys";
						regulator-always-on;
						regulator-min-microvolt = <0x10c8e0>;
						regulator-max-microvolt = <0x10c8e0>;
					};

					dldo1 {
						regulator-name = "vcc-hdmi";
						regulator-min-microvolt = <0x325aa0>;
						regulator-max-microvolt = <0x325aa0>;
						phandle = <0x04>;
					};

					dldo2 {
						regulator-name = "vcc-mipi";
						regulator-min-microvolt = <0x325aa0>;
						regulator-max-microvolt = <0x325aa0>;
					};

					dldo3 {
						regulator-name = "dldo3";
					};

					dldo4 {
						regulator-name = "vcc-wifi";
						regulator-min-microvolt = <0x325aa0>;
						regulator-max-microvolt = <0x325aa0>;
					};

					eldo1 {
						regulator-name = "cpvdd";
						regulator-min-microvolt = <0x1b7740>;
						regulator-max-microvolt = <0x1b7740>;
						phandle = <0x44>;
					};

					eldo2 {
						regulator-name = "eldo2";
					};

					eldo3 {
						regulator-name = "eldo3";
					};

					fldo1 {
						regulator-name = "vcc-1v2-hsic";
						regulator-min-microvolt = <0x124f80>;
						regulator-max-microvolt = <0x124f80>;
					};

					fldo2 {
						regulator-name = "vdd-cpus";
						regulator-always-on;
						regulator-min-microvolt = <0x10c8e0>;
						regulator-max-microvolt = <0x10c8e0>;
					};

					ldo-io0 {
						regulator-name = "ldo-io0";
						status = "disabled";
					};

					ldo-io1 {
						regulator-name = "ldo-io1";
						status = "disabled";
					};

					rtc-ldo {
						regulator-always-on;
						regulator-min-microvolt = <0x2dc6c0>;
						regulator-max-microvolt = <0x2dc6c0>;
						regulator-name = "vcc-rtc";
					};

					drivevbus {
						regulator-name = "drivevbus";
						status = "disabled";
					};
				};

				usb-power {
					compatible = "x-powers,axp803-usb-power-supply", "x-powers,axp813-usb-power-supply";
					status = "disabled";
				};
			};
		};
	};

	opp-table-cpu {
		compatible = "operating-points-v2";
		opp-shared;
		phandle = <0x06>;

		opp-648000000 {
			opp-hz = <0x00 0x269fb200>;
			opp-microvolt = <0xfde80>;
			clock-latency-ns = <0x3b9b0>;
		};

		opp-816000000 {
			opp-hz = <0x00 0x30a32c00>;
			opp-microvolt = <0x10c8e0>;
			clock-latency-ns = <0x3b9b0>;
		};

		opp-912000000 {
			opp-hz = <0x00 0x365c0400>;
			opp-microvolt = <0x111700>;
			clock-latency-ns = <0x3b9b0>;
		};

		opp-960000000 {
			opp-hz = <0x00 0x39387000>;
			opp-microvolt = <0x11b340>;
			clock-latency-ns = <0x3b9b0>;
		};

		opp-1008000000 {
			opp-hz = <0x00 0x3c14dc00>;
			opp-microvolt = <0x124f80>;
			clock-latency-ns = <0x3b9b0>;
		};

		opp-1056000000 {
			opp-hz = <0x00 0x3ef14800>;
			opp-microvolt = <0x12ebc0>;
			clock-latency-ns = <0x3b9b0>;
		};

		opp-1104000000 {
			opp-hz = <0x00 0x41cdb400>;
			opp-microvolt = <0x1339e0>;
			clock-latency-ns = <0x3b9b0>;
		};

		opp-1152000000 {
			opp-hz = <0x00 0x44aa2000>;
			opp-microvolt = <0x13d620>;
			clock-latency-ns = <0x3b9b0>;
		};
	};

	aliases {
		ethernet0 = "/soc/ethernet@1c30000";
		serial0 = "/soc/serial@1c28000";
		serial1 = "/soc/serial@1c28400";
		serial2 = "/soc/serial@1c28800";
		serial3 = "/soc/serial@1c28c00";
		serial4 = "/soc/serial@1c29000";
	};

	hdmi-connector {
		compatible = "hdmi-connector";
		type = "a";

		port {

			endpoint {
				remote-endpoint = <0x49>;
				phandle = <0x42>;
			};
		};
	};
};
```