<template>
    <div>
        <span>0：{{OriArr}}</span>
        <span>1：{{newArr}}</span>
    </div>
</template>

<script>
export default {
    data(){
        return{
           OriArr:[1,2,1,2,3,5,1],
           newArr:[] 
        }
    },
    methods:{
        compute(){
            this.OriArr.forEach(item=>{
                let i = this.newArr.findIndex((n) => n == item)
                if(i<0){
                    this.newArr.push(item)
                }
            })
        }
    },
    mounted(){
        this.compute()
    }
}
</script>
