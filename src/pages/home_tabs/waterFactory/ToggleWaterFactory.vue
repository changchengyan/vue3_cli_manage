<template>
    <div id="toggleWaterFactory">
        <component :is="componentModule"  ref="factory_monitor" :factoryId="factoryId" :factoryName="factoryName"></component>
    </div>
</template>

<script>

    const PLANE_MODULE = ()=>import('./WaterFactoryPlane');
    const ZHUTAI_MODULE = ()=>import('./Zhutai');
    const VIDEO_MODULE = ()=>import('./WaterFactoryVideo');
    const VIEWRECORD_MODULE = ()=>import('./FactoryViewRecord');
    const BASEMSG_MODULE = ()=>import('./FactoryBaseMsg');


    export default {
        name: 'toggleWaterFactory',
        props:{
            factoryId:{
                type:String,
                defaultProps:''
            },
            factoryName:{
                type:String,
                defaultProps:''
            }
        },
        data() {
            return {
                activeComponentIndex: 0,
                videoList: [],
                staticPath: '',
                componentModule:PLANE_MODULE
            };
        },
        methods: {
            activeThis(index) {
                this.activeComponentIndex = index;
                switch (index) {
                    case 0:
                        this.componentModule = PLANE_MODULE;
                        break;
                    case 1:
                        this.componentModule = ZHUTAI_MODULE;
                        break;
                    case 2:
                        this.componentModule = VIDEO_MODULE;
                        break;
                    case 3:
                        this.componentModule = VIEWRECORD_MODULE;
                        break;
                    case 4:
                        this.componentModule = BASEMSG_MODULE;
                        break;

                }
            },
            resize() {
                let that = this;
                this.$nextTick(() => {
                    if (that.activeComponentIndex == 1) {
                        that.$refs.factory_monitor.resize();
                    }
                });
            }
        },
        created() {
            this.staticPath = window.DSE.static_baseUrl;
            console.log(this.factoryId);
            console.log('_________________');
        },
        beforeDestroy() {
            this.activeComponentIndex = 0;
        }
    };
</script>

<style scoped lang="scss">
    #toggleWaterFactory {
        width: 100%;
        height: 100%;

        img {
            width: 100%;
        }
    }
</style>
