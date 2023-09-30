import { LightningElement,api,track,wire } from 'lwc';
import getRoles from '@salesforce/apex/SoftwareDevelopmentScopeCtrl.getRoles';
import getScopeAndServiceDetail from '@salesforce/apex/SoftwareDevelopmentScopeCtrl.getScopeAndServiceDetail';
import {ShowToastEvent} from 'lightning/platformShowToastEvent';
import {updateRecord,getRecord} from 'lightning/uiRecordApi';
import JSON_DATA from '@salesforce/schema/Scope__c.JSON_Data__c';
import SCOPE_NAME from '@salesforce/schema/Scope__c.Scope_Name__c';
import ID_FIELD from '@salesforce/schema/Scope__c.Id';
import createActions from '@salesforce/apex/ContentMarketingCtrl.createActions';
import { reduceErrors } from 'c/ldsUtils';
import userId from '@salesforce/user/Id';
import ADMIN_SCOPING from '@salesforce/schema/User.Admin_Scoping__c';

export default class ContentMarketingScope extends LightningElement {
    @api recordId;
    @track contentMarketing={};
    @track sessionData = [];
    @track bloggingSessionData = [];
    @track socPostingSessionData = [];
    @track emailMarketingSessionData = [];
    @track trainingSessionData=[];
    showSpinner=true;
    mailChimpRowNames=['MailChimp']
    campaignTypes = [{
        label: 'Drip',
        value: 'Drip'
    }, {
        label: 'Newsletter',
        value: 'Newsletter'
    },{
        label: 'Drip(Text-only)',
        value: 'Drip_Text_Only'
    },{
        label: 'Re-engagement',
        value: 'Re_engagement'
    }];
    radioOptions = [{
        label: 'Yes',
        value: 'Yes'
    }, {
        label: 'No',
        value: 'No'
    }];
    profileOptions = [{
        label: 'Create New',
        value: 'Create_New'
    }, {
        label: 'Update Existing',
        value: 'Update_Existing'
    },{
        label: 'No',
        value: 'No'
    },
];
    @track services = [{
        label: 'Blogging',
        value: 'Blogging'
    }, {
        label: 'Email Marketing',
        value: 'Email_Marketing'
    },{
        label: 'Social Posting',
        value: 'Social_Posting'
    },{
        label: 'Review Management',
        value: 'Review_Management'
    }];
    @track trainingTypes = [{
        label: 'Blog',
        value: 'Blog'
    }, {
        label: 'Email',
        value: 'Email'
    },{
        label: 'LinkedIn',
        value: 'LinkedIn'
    },{
        label: 'Review Management',
        value: 'Review Management'
    },{
        label: 'Social',
        value: 'Social'
    },{
        label: 'Other',
        value: 'Other'
    }];
    selectedServices = [];
    showSelectService=false;
    Blogging = false;
    EmailMarketing = false;
    ReviewManagement = false;

    SocialPosting=false;
    regularRowNames=['Write:Regular','Post:Regular','Share:Regular'];
    hoursPerMonthRowsName=['Hours'];

    miniRowNames=['Write:Mini','Post:Mini','Share:Mini'];
    longRowNames=['Write:Long','Post:Long','Share:Long'];
    editedRowNames=['Write:Edited_SEOed','Post:Edited_SEOed','Share:Edited_SEOed'];
    bloggingRowNamesTemp=[];
    monthlyStartDate = new Date("2022-04-05");
    monthlyEndDate = new Date("2022-12-05");
    @track socialPlatforms = [{
        label: 'Facebook',
        value: 'Facebook'
    },{
        label: 'Google my business',
        value: 'Google_my_business'
    },{
        label: 'Instagram',
        value: 'Instagram'
    },{
        label: 'LinkedIn',
        value: 'LinkedIn'
    },{
        label: 'Twitter',
        value: 'Twitter'
    }];
    @track bloggingTypes = [{
        label: 'Mini',
        value: 'Mini'
    }, {
        label: 'Regular',
        value: 'Regular'
    },{
        label: 'Long',
        value: 'Long'
    },
    {
        label: 'Edited/SEOed',
        value: 'Edited_SEOed'
    }];
    @track socialPostingPlatforms = [{
        label: 'Twitter',
        value: 'Twitter'
    }, {
        label: 'Google My Business',
        value: 'Google_My_Business'
    },{
        label: 'LinkedIn',
        value: 'LinkedIn'
    },
    {
        label: 'Instagram',
        value: 'Instagram'
    }];
    @track selectedSocialProfileTypes=[];
    @track socialProfileTypes = [{
        label: 'Facebook',
        value: 'Facebook'
    }, {
        label: 'Instagram',
        value: 'Instagram'
    },{
        label: 'LinkedIn',
        value: 'LinkedIn'
    },
    {
        label: 'Twitter',
        value: 'Twitter'
    },{
        label: 'YouTube',
        value: 'YouTube'
    }];
    communityMgtRowNames=['Community Management'];
    postsRowNames=['number of Posts'];
    @track themes=[{label:'Blog Articles linking to website',value:'Blog_Articles_linking_to_website'},
    {label:'Community News/Events',value:'Community_News_Events'},
    {label:'Engagement',value:'Engagement'},
    {label:'Industry News',value:'Industry_News'},
    {label:'Polls',value:'Polls'},
    {label:'Reviews',value:'Reviews'},
    {label:'Tips',value:'Tips'},
    {label:'Trivia',value:'Trivia'},]

    facebook=false;
    twitter=false;
    linkedIn=false;
    instagram=false;
    youtube=false;
    showEMTables=false;
    @track selectedSocialPostingPlatforms=[];
    @track selectedSocialPostingThemes=[];
    @track selectedPlatforms=[];
    @track selectedBlogTypes=[];
    @track selectedCampaignTypes=[];
    @track selectedTrainingTypes=[];
    DRIP;NEWSLETTER;REENGAGEMENT;DRIPTEXT;
    emarketingRowNames=[];
    @track emarketingRowTemp=[];
    @track dripRowNamesTemp=[];
    dripRowNames=[];

    @track dripTextRowNames=[];
    @track dripTextOnlyRowNames=[];
    @track newletterRowNames=[];
    @track reengagementRowNames=[];

    @track hoursPerMonthForManagement=[];

    closedWonOpp = false;
    adminScoping = false;
    
    @wire(getRecord, { recordId: userId, fields: [ADMIN_SCOPING]}) 
    userDetails({error, data}) {
        if (data) {
            if(data.fields.Admin_Scoping__c.value == true){
                this.adminScoping = true;
            }
        } else if (error) {
            console.log(error);
        }
    }

    connectedCallback() {
        this.getRolesFromBackend();
    }

    showEmarketingTable(){
        if(this.emarketingRowNames.length>0){
            return true;
        }else{
            return false;
        }
    }

    showMiniMonthlyData=false;
    showRegularMonthlyData=false;
    showLongMonthlyData=false;
    showEditedMonthlyData=false;


    handleBlogginTypeChange(event){
        //console.log(JSON.stringify(event));
        let rowNames=event.detail.payload.values;

        this.setTypesPosted(rowNames);
        
        /*
        console.log(rowNames);
        this.bloggingRowNamesTemp=rowNames;
        this.bloggingRowNames=[];

        for(const type of this.bloggingRowNamesTemp){
            this.bloggingRowNames.push('Write:'+type);
            this.bloggingRowNames.push('Post:'+type);
            this.bloggingRowNames.push('Share:'+type);
        }
        */
    }

    setTypesPosted(selectedTypes){
        if(selectedTypes.includes('Mini')){
            this.showMiniMonthlyData=true;
        }else{
            this.showMiniMonthlyData=false;
        }

        if(selectedTypes.includes('Regular')){
            this.showRegularMonthlyData=true;
        }else{
            this.showRegularMonthlyData=false;
        }

        if(selectedTypes.includes('Long')){
            this.showLongMonthlyData=true;
        }else{
            this.showLongMonthlyData=false;
        }

        if(selectedTypes.includes('Edited_SEOed')){
            this.showEditedMonthlyData=true;
        }else{
            this.showEditedMonthlyData=false;
        }
    }

    setSocialProfileTypes(profilePages){
        if(profilePages.includes('Facebook')){
            this.facebook=true;
       }else{
            this.facebook=false;
       }
       if(profilePages.includes('Instagram')){
            this.instagram=true;
       }else{
            this.instagram=false;
       }
       if(profilePages.includes('LinkedIn')){
            this.linkedIn=true;
       }else{
            this.linkedIn=false;
       }
       if(profilePages.includes('Twitter')){
            this.twitter=true;
       }else{
            this.twitter=false;
       }
       
       if(profilePages.includes('YouTube')){
            this.youtube=true;
       }else{
            this.youtube=false;
       }
    }
    
    handleProfilePageChange(event){
        let profilePages=event.detail.payload.values;
        console.log(profilePages);

       this.setSocialProfileTypes(profilePages);
    }

    handleSessionCountChange(event) {
        if (event.detail.value) {
            let changedValue = event.detail.value;
            console.log(changedValue);
            if (this.sessionData.length > 0) {
                let actualArrLen = this.sessionData.length;
                console.log(actualArrLen);
                console.log(changedValue - actualArrLen);
                if (changedValue - actualArrLen > 0) {
                    for (let index = 1; index <= changedValue - actualArrLen; index++) {
                        let session = {};
                        session.contentSessionDuration = '';
                        session.contentSessionPerson = [];
                        session.contentSessionDate = '';
                        this.sessionData.push(session);
                    }
                } else if (changedValue - actualArrLen < 0) {
                    for (let index = 1; index <= (changedValue - actualArrLen) * -1; index++) {
                        this.sessionData.pop();
                    }
                }
            } else {
                this.sessionData = [];
                for (let index = 1; index <= changedValue; index++) {
                    let session = {};
                    session.contentSessionDuration = '';
                    session.contentSessionPerson = [];
                    session.contentSessionDate = '';
                    this.sessionData.push(session);

                }
            }
        }
    }

    

    handleTrainingCountChange(event){
        if (event.detail.value) {
            let changedValue = event.detail.value;
            
            if (this.trainingSessionData.length > 0) {
                let actualArrLen = this.trainingSessionData.length;
                
                if (changedValue - actualArrLen > 0) {
                    for (let index = 1; index <= changedValue - actualArrLen; index++) {
                        let session = {};
                        session.trainingSessionDuration = '';
                        session.trainingSessionPerson = [];
                        session.trainingSessionDate = '';
                        this.trainingSessionData.push(session);
                    }
                } else if (changedValue - actualArrLen < 0) {
                    for (let index = 1; index <= (changedValue - actualArrLen) * -1; index++) {
                        this.trainingSessionData.pop();
                    }
                }
            } else {
                this.trainingSessionData = [];
                for (let index = 1; index <= changedValue; index++) {
                    let session = {};
                    session.trainingSessionDuration = '';
                    session.trainingSessionPerson = [];
                    session.trainingSessionDate = '';
                    this.trainingSessionData.push(session);

                }
            }
        }
    }

    handleBloggingCountChange(event){
        if (event.detail.value) {
            let changedValue = event.detail.value;
            
            if (this.bloggingSessionData.length > 0) {
                let actualArrLen = this.bloggingSessionData.length;
                
                if (changedValue - actualArrLen > 0) {
                    for (let index = 1; index <= changedValue - actualArrLen; index++) {
                        let session = {};
                        session.contentSessionDuration = '';
                        session.prepSessionDuration= '';
                        session.contentSessionPerson = [];
                        session.contentSessionDate = '';
                        this.bloggingSessionData.push(session);
                    }
                } else if (changedValue - actualArrLen < 0) {
                    for (let index = 1; index <= (changedValue - actualArrLen) * -1; index++) {
                        this.bloggingSessionData.pop();
                    }
                }
            } else {
                this.bloggingSessionData = [];
                for (let index = 1; index <= changedValue; index++) {
                    let session = {};
                    session.contentSessionDuration = '';
                    session.prepSessionDuration= '';
                    session.contentSessionPerson = [];
                    session.contentSessionDate = '';
                    this.bloggingSessionData.push(session);

                }
            }
        }
    }

    handleBlogginSessionDataChange(event) {
        var rowIndex = event.currentTarget.dataset.index;
        console.log(rowIndex);
        console.log(event.target.name);

        if (event.target.name === 'contentSessionDuration') {
            this.bloggingSessionData[rowIndex].contentSessionDuration = event.target.value;
        }else if(event.target.name === 'prepSessionDuration'){
            this.bloggingSessionData[rowIndex].prepSessionDuration = event.target.value;
        } else if (event.target.name === 'contentSessionDate') {
            this.bloggingSessionData[rowIndex].contentSessionDate = event.target.value;
        } else if (event.target.name === 'contentSessionPerson') {
            this.bloggingSessionData[rowIndex].contentSessionPerson = event.detail.payload.values;
        }
    }

    handleSocialPostingCountChange(event){
        if (event.detail.value) {
            let changedValue = event.detail.value;
            
            if (this.socPostingSessionData.length > 0) {
                let actualArrLen = this.socPostingSessionData.length;
                
                if (changedValue - actualArrLen > 0) {
                    for (let index = 1; index <= changedValue - actualArrLen; index++) {
                        let session = {};
                        session.contentSessionDuration = '';
                        session.prepSessionDuration = '';
                        session.contentSessionPerson = [];
                        session.contentSessionDate = '';
                        this.socPostingSessionData.push(session);
                    }
                } else if (changedValue - actualArrLen < 0) {
                    for (let index = 1; index <= (changedValue - actualArrLen) * -1; index++) {
                        this.socPostingSessionData.pop();
                    }
                }
            } else {
                this.socPostingSessionData = [];
                for (let index = 1; index <= changedValue; index++) {
                    let session = {};
                    session.contentSessionDuration = '';
                    session.prepSessionDuration = '';
                    session.contentSessionPerson = [];
                    session.contentSessionDate = '';
                    this.socPostingSessionData.push(session);

                }
            }
        }
    }

    handleSocPostingSessionDataChange(event) {
        var rowIndex = event.currentTarget.dataset.index;
        console.log(rowIndex);
        console.log(event.target.name);

        if (event.target.name === 'contentSessionDuration') {
            this.socPostingSessionData[rowIndex].contentSessionDuration = event.target.value;
        } else if(event.target.name === 'prepSessionDuration'){
            this.socPostingSessionData[rowIndex].prepSessionDuration = event.target.value;
        } else if (event.target.name === 'contentSessionDate') {
            this.socPostingSessionData[rowIndex].contentSessionDate = event.target.value;
        } else if (event.target.name === 'contentSessionPerson') {
            this.socPostingSessionData[rowIndex].contentSessionPerson = event.detail.payload.values;
        }
    }

    handleEmailMarketingCountChange(event){
        if (event.detail.value) {
            let changedValue = event.detail.value;
            
            if (this.emailMarketingSessionData.length > 0) {
                let actualArrLen = this.emailMarketingSessionData.length;
                
                if (changedValue - actualArrLen > 0) {
                    for (let index = 1; index <= changedValue - actualArrLen; index++) {
                        let session = {};
                        session.contentSessionDuration = '';
                        session.prepSessionDuration = '';
                        session.contentSessionPerson = [];
                        session.contentSessionDate = '';
                        this.emailMarketingSessionData.push(session);
                    }
                } else if (changedValue - actualArrLen < 0) {
                    for (let index = 1; index <= (changedValue - actualArrLen) * -1; index++) {
                        this.emailMarketingSessionData.pop();
                    }
                }
            } else {
                this.emailMarketingSessionData = [];
                for (let index = 1; index <= changedValue; index++) {
                    let session = {};
                    session.contentSessionDuration = '';
                    session.prepSessionDuration = '';
                    session.contentSessionPerson = [];
                    session.contentSessionDate = '';
                    this.emailMarketingSessionData.push(session);

                }
            }
        }
    }

    handleEmailMarketingSessionDataChange(event) {
        var rowIndex = event.currentTarget.dataset.index;
        console.log(rowIndex);
        console.log(event.target.name);

        if (event.target.name === 'contentSessionDuration') {
            this.emailMarketingSessionData[rowIndex].contentSessionDuration = event.target.value;
        } else if(event.target.name === 'prepSessionDuration'){
            this.emailMarketingSessionData[rowIndex].prepSessionDuration = event.target.value;
        }else if (event.target.name === 'contentSessionDate') {
            this.emailMarketingSessionData[rowIndex].contentSessionDate = event.target.value;
        } else if (event.target.name === 'contentSessionPerson') {
            this.emailMarketingSessionData[rowIndex].contentSessionPerson = event.detail.payload.values;
        }
    }

    handleSessionDataChange(event) {
        var rowIndex = event.currentTarget.dataset.index;
        console.log(rowIndex);
        console.log(event.target.name);

        if (event.target.name === 'contentSessionDuration') {
            this.sessionData[rowIndex].contentSessionDuration = event.target.value;
        } else if (event.target.name === 'contentSessionDate') {
            this.sessionData[rowIndex].contentSessionDate = event.target.value;
        } else if (event.target.name === 'contentSessionPerson') {
            this.sessionData[rowIndex].contentSessionPerson = event.detail.payload.values;
        }
    }

    handleTrainingDataChange(event) {
        var rowIndex = event.currentTarget.dataset.index;
        console.log(rowIndex);
        console.log(event.target.name);

        if (event.target.name === 'trainingSessionDuration') {
            this.trainingSessionData[rowIndex].trainingSessionDuration = event.target.value;
        } else if (event.target.name === 'trainingSessionDate') {
            this.trainingSessionData[rowIndex].trainingSessionDate = event.target.value;
        } else if (event.target.name === 'trainingSessionPerson') {
            this.trainingSessionData[rowIndex].trainingSessionPerson = event.detail.payload.values;
        }
    }

    getRolesFromBackend() {
        getRoles()
            .then(result => {
                this.roles = [];
                console.log(result);

                for (const role of result) {
                    this.roles.push({
                        label: role.Name,
                        value: role.Id
                    });
                }
                
                this.getScopeFromBackend();
            }).catch(error => {
                console.log(error);
                this.showSpinner = false;
            })
    }

    serviceDet;
    getScopeFromBackend(){
        getScopeAndServiceDetail({
            recordId: this.recordId
        })
        .then(result => {
            let scopeRecord=result.scopeRecord;
            let serviceDetail=result.serviceDetailRecord;
            this.serviceDet=serviceDetail;

            if(scopeRecord.Opportunity__r.StageName=='Closed Won'){
                this.closedWonOpp = true;
            }

            if (scopeRecord) {
                if (scopeRecord.Opportunity__r && scopeRecord.Opportunity__r.MAP_Start_Date__c) {
                    this.monthlyStartDate = scopeRecord.Opportunity__r.MAP_Start_Date__c;
                } else {
                    this.monthlyStartDate = new Date();
                }
                if (scopeRecord.Opportunity__r && scopeRecord.Opportunity__r.MAP_End_Date__c) {
                    this.monthlyEndDate = scopeRecord.Opportunity__r.MAP_End_Date__c;
                } else {
                    this.monthlyEndDate = new Date(new Date().getFullYear(), 11, 31);
                }
            }
            if (scopeRecord && scopeRecord.JSON_Data__c) {
                this.contentMarketing = JSON.parse(scopeRecord.JSON_Data__c);

                this.contentMarketing.scopeName = scopeRecord.Scope_Name__c;
                console.log('Scope Name : '+this.contentMarketing.scopeName);


                if(this.contentMarketing.bloggingRowNames){
                    this.bloggingRowNames=this.contentMarketing.bloggingRowNames;
                }
                if(this.contentMarketing.dripTextRowNames){
                    this.dripTextRowNames=this.contentMarketing.dripTextRowNames;
                }
                if(this.contentMarketing.dripTextOnlyRowNames){
                    this.dripTextOnlyRowNames=this.contentMarketing.dripTextOnlyRowNames;
                }
                if(this.contentMarketing.dripTextOnlyOptMonthlyData){
                    this.dripTextOnlyOptMonthlyData=this.contentMarketing.dripTextOnlyOptMonthlyData;
                }
                if(this.contentMarketing.hoursPerMonthForManagement){
                    this.hoursPerMonthForManagement = this.contentMarketing.hoursPerMonthForManagement;
                }
                if(this.contentMarketing.newletterRowNames){
                    this.newletterRowNames=this.contentMarketing.newletterRowNames;
                }
                if(this.contentMarketing.reengagementRowNames){
                    this.reengagementRowNames=this.contentMarketing.reengagementRowNames;
                }

                if(this.dripTextRowNames.length>0 || this.dripTextOnlyRowNames.length>0 || this.newletterRowNames.length>0 || this.reengagementRowNames.length>0){
                    this.showEMTables=true;
                }else{
                    this.showEMTables=false;
                }
                /*
                if(this.contentMarketing.sessionData){
                    this.sessionData=this.contentMarketing.sessionData;
                }*/
                if(this.contentMarketing.socPostingSessionData){
                    this.socPostingSessionData=this.contentMarketing.socPostingSessionData;
                }
                if(this.contentMarketing.emailMarketingSessionData){
                    this.emailMarketingSessionData=this.contentMarketing.emailMarketingSessionData;
                }
                if(this.contentMarketing.bloggingSessionData){
                    this.bloggingSessionData=this.contentMarketing.bloggingSessionData;
                }
                if(this.contentMarketing.trainingSessionData){
                    this.trainingSessionData=this.contentMarketing.trainingSessionData;
                }
                if(this.contentMarketing.trainingType){
                    this.selectedTrainingTypes=this.contentMarketing.trainingType;
                }
                if (this.contentMarketing.Blogging) {
                    this.selectedServices.push('Blogging');
                    this.Blogging = true;

                    if(this.contentMarketing.Blogging.bloggingTypes){
                        this.selectedBlogTypes=this.contentMarketing.Blogging.bloggingTypes;
                        this.setTypesPosted(this.selectedBlogTypes);
                    }
                    if(this.contentMarketing.Blogging.socialPlatforms){
                        this.selectedPlatforms=this.contentMarketing.Blogging.socialPlatforms;
                    }
                }
                if (this.contentMarketing.EmailMarketing) {
                    console.log('EmailMarketing');
                    this.selectedServices.push('Email_Marketing');
                    this.EmailMarketing = true;

                    if(this.contentMarketing.EmailMarketing.campaignTypes){
                        this.selectedCampaignTypes=this.contentMarketing.EmailMarketing.campaignTypes;
                        if(this.selectedCampaignTypes.includes('Drip')){
                            this.DRIP=true;
                        }
                        if(this.selectedCampaignTypes.includes('Newsletter')){
                            this.NEWSLETTER=true;
                        }
                        if(this.selectedCampaignTypes.includes('Drip_Text_Only')){
                            this.DRIPTEXT=true;
                        }
                        if(this.selectedCampaignTypes.includes('Re_engagement')){
                            this.REENGAGEMENT=true;
                        }
                    }
                }
                if (this.contentMarketing.SocialPosting) {
                    console.log('SocialPosting');
                    this.selectedServices.push('Social_Posting');
                    this.SocialPosting = true;

                    if(this.contentMarketing.SocialPosting.socialPostingThemes){
                        this.selectedSocialPostingThemes=this.contentMarketing.SocialPosting.socialPostingThemes;
                    }
                    if(this.contentMarketing.SocialPosting.socialPostingPlatform){
                        this.selectedSocialPostingPlatforms=this.contentMarketing.SocialPosting.socialPostingPlatform;
                    }

                    if(this.contentMarketing.SocialPosting.socialProfileTypes){
                        this.selectedSocialProfileTypes=this.contentMarketing.SocialPosting.socialProfileTypes;
                        this.setSocialProfileTypes(this.selectedSocialProfileTypes);
                    }
                }
                if (this.contentMarketing.ReviewManagement) {
                    console.log('ReviewManagement');
                    debugger;
                    this.selectedServices.push('Review_Management');
                    this.ReviewManagement = true;
                }
            }
            this.showSpinner = false;
            this.showSelectService = true;
        })
        .catch(error => {
            console.log(error);
            this.showSpinner = false;
        })
    }

    get trainingSession() {
        if (this.contentMarketing.trainingSession == 'Yes') {
            return true;
        } else {
            return false;
        }
    }

    get blogCategories() {
        if (this.contentMarketing.Blogging.blogCategories == 'Yes') {
            return true;
        } else {
            return false;
        }
    }

    get clientPlatformSharingIncharge() {
        console.log(this.contentMarketing.Blogging.clientPlatformSharingIncharge);
        if (this.contentMarketing.Blogging.clientPlatformSharingIncharge == 'Yes') {
            
            return true;
        } else {
            return false;
        }
    }

    get domainValidated() {
        if (this.contentMarketing.EmailMarketing.domainValidated == 'No') {
            return true;
        } else {
            return false;
        }
    }

    get socialProfilePages() {
        if (this.contentMarketing.SocialPosting.socialProfile == 'Yes') {
            return true;
        } else {
            return false;
        }
    }

    get additionalNetworks() {
        if (this.contentMarketing.SocialPosting.additionalNetworks == 'Yes') {
            return true;
        } else {
            return false;
        }
    }

    get notMailChimp() {
        if (this.contentMarketing.EmailMarketing.mailChimpProfile == 'No') {
            return false;
        } else {
            return true;
        }
    }

    get notNewProfile() {
        if (this.contentMarketing.ReviewManagement.createNewProfileAccount == 'No') {
            return false;
        } else {
            return true;
        }
    }

    handleChange(event) {
        let nodeName = event.target.name;
        console.log(nodeName);
        console.log(event.target.value);

        if (nodeName == 'trainingSession') {
            this.contentMarketing.trainingSession = event.target.value;
        } else if (nodeName == 'blogCategories') {
            this.contentMarketing.Blogging.blogCategories = event.target.value;
        } else if (nodeName == 'clientPlatformSharingIncharge') {
            this.contentMarketing.Blogging.clientPlatformSharingIncharge = event.target.value;
        } else if (nodeName == 'domainValidated') {
            this.contentMarketing.EmailMarketing.domainValidated = event.target.value;
        } else if (nodeName == 'socialProfile') {
            this.contentMarketing.SocialPosting.socialProfile = event.target.value;
        }else if (nodeName == 'additionalNetworks') {
            this.contentMarketing.SocialPosting.additionalNetworks = event.target.value;
        }else if (nodeName == 'mailChimpProfile') {
            this.contentMarketing.EmailMarketing.mailChimpProfile = event.target.value;
        }else if (nodeName == 'createNewProfileAccount') {
            this.contentMarketing.ReviewManagement.createNewProfileAccount = event.target.value;
        }
    }



    handleMultiSelect(event) {
        console.log(JSON.stringify(event));

        if (event.detail.payload.name == 'service') {
            console.log(event.detail.payload.name);
            let currentValues = event.detail.payload.values;
            console.log(currentValues);
            if (this.contentMarketing.Blogging && !currentValues.includes('Blogging')) {
                delete this.contentMarketing.Blogging;
                this.Blogging = false
            } else if (!this.contentMarketing.Blogging && currentValues.includes('Blogging')) {
                this.contentMarketing['Blogging'] = {};
                this.contentMarketing.Blogging.bloggingStrategyNotes=this.serviceDet['Blogging'];
                this.Blogging = true
            }

            if (this.contentMarketing.EmailMarketing && !currentValues.includes('Email_Marketing')) {
                delete this.contentMarketing.EmailMarketing;
                this.EmailMarketing = false;
            } else if (!this.contentMarketing.EmailMarketing && currentValues.includes('Email_Marketing')) {
                this.contentMarketing['EmailMarketing'] = {};
                this.contentMarketing.EmailMarketing.emarketingStrategyNotes=this.serviceDet['Email Marketing'];
                this.EmailMarketing = true;
            }

            if (this.contentMarketing.SocialPosting && !currentValues.includes('Social_Posting')) {
                delete this.contentMarketing.SocialPosting;
                this.SocialPosting = false;
            } else if (!this.contentMarketing.SocialPosting && currentValues.includes('Social_Posting')) {
                this.contentMarketing['SocialPosting'] = {};
                this.contentMarketing.SocialPosting.socialPostingStrategyNotes=this.serviceDet['Social Posting'];
                this.SocialPosting = true;
            }

            if (this.contentMarketing.ReviewManagement && !currentValues.includes('Review_Management')) {
                delete this.contentMarketing.ReviewManagement;
                this.ReviewManagement = false;
            } else if (!this.contentMarketing.ReviewManagement && currentValues.includes('Review_Management')) {
                this.contentMarketing['ReviewManagement'] = {};
                this.contentMarketing.ReviewManagement.reviewManagementStrategyNotes=this.serviceDet['Review Management'];
                this.ReviewManagement = true;
            }

            if (!currentValues.length) {
                this.Blogging = false;
                this.EmailMarketing = false;
                this.SocialPosting=false;
            }
        }

    }
    

    handleCampaignTypeChange(event){
        this.selectedCampaignTypes=[];
        this.DRIP=false;
        this.NEWSLETTER=false;
        this.DRIPTEXT=false;
        this.REENGAGEMENT=false;


        if (event.detail.payload.name == 'campaignTypes') {
            console.log(event.detail.payload.name);
            let currentValues = event.detail.payload.values;
            console.log(currentValues);
            if (currentValues.includes('Drip')) {
                this.DRIP=true;
                this.selectedCampaignTypes.push('Drip');
            }else{
                this.DRIP=false;
            }
            if (currentValues.includes('Newsletter')) {
                this.NEWSLETTER=true;
                this.selectedCampaignTypes.push('Newsletter');
            }if (currentValues.includes('Drip_Text_Only')) {
                this.DRIPTEXT=true;
                this.selectedCampaignTypes.push('Drip_Text_Only');
            }else{
                this.DRIPTEXT=false;
            }
            if (currentValues.includes('Re_engagement')) {
                this.REENGAGEMENT=true;
                this.selectedCampaignTypes.push('Re_engagement');
            }else{
                this.REENGAGEMENT=false;
            }
        }
    }

    

    HandleCampaignFieldChange(event){
        this.emarketingRowTemp=[];
        this.emarketingRowNames=[];
        this.dripRowNames=[];
        let emailMarketing={};
        this.dripTextRowNames=[];
        this.dripTextOnlyRowNames=[];
        this.newletterRowNames=[];
        this.reengagementRowNames=[];

        let allinputs = this.template.querySelectorAll('[data-id="emailMarketing"]');
        //let dripTxtMonthData = this.template.querySelectorAll('c-dynamic-multi-input-grid');
      

        for (const ip of allinputs) {
            if(ip.name=='dripCampaignCount'){
                /*
                if(!this.contentMarketing.EmailMarketing.dripTextMonthlyData && dripTxtMonthData){
                    for (const mi of dripTxtMonthData) {
                        if(mi.name=='dripTextMonthlyData'){
                            this.contentMarketing.EmailMarketing.dripTextMonthlyData = mi.monthlyData();
                        }
                    }
                }*/
                
                for (let index = 0; index < ip.value; index++) {
                    this.dripTextRowNames.push('Drip'+index);
                }
            }
            if(ip.name=='dripTextCampaignCount'){
                for (let index = 0; index < ip.value; index++) {
                    this.dripTextOnlyRowNames.push('Drip(Text-Only)'+index);
                }
                this.dripRowNamesTemp['driptext']='test';
            }
            if(ip.name=='newsletterCampaignCount'){
                for (let index = 0; index < ip.value; index++) {
                    this.newletterRowNames.push('Newsletter'+index);
                }
            }
            if(ip.name=='reengagementCampaignCount'){
                for (let index = 0; index < ip.value; index++) {
                    this.reengagementRowNames.push('Re-engagement'+index);
                }
            }
        }

        if(this.dripTextRowNames.length>0 || this.dripTextOnlyRowNames.length>0 || this.newletterRowNames.length>0 || this.reengagementRowNames.length>0){
            this.showEMTables=true;
        }else{
            this.showEMTables=false;
        }

        console.log('this.newletterRowNames'+this.newletterRowNames);
    }

    isInputValid() {
        let isValid = true;
        let inputFields = this.template.querySelectorAll('lightning-input');
        inputFields.forEach(inputField => {
            if (!inputField.checkValidity()) {
                inputField.reportValidity();
                isValid = false;
                this.dispatchEvent(
                    new ShowToastEvent({
                        title: 'Error',
                        message: 'Please select required fields',
                        variant: 'error'
                    })
                );
            }
        });
        return isValid;
    }

    isComboValid() {
        let isValid = true;
        let inputFields = this.template.querySelectorAll('lightning-combobox');
        inputFields.forEach(inputField => {
            if (!inputField.checkValidity()) {
                inputField.reportValidity();
                isValid = false;
                this.dispatchEvent(
                    new ShowToastEvent({
                        title: 'Error',
                        message: 'Please select required fields',
                        variant: 'error'
                    })
                );
            }
        });
        return isValid;
    }

    isRadioValid() {
        let isValid = true;
        let inputFields = this.template.querySelectorAll('lightning-radio-group');
        inputFields.forEach(inputField => {
            if (!inputField.checkValidity()) {
                inputField.reportValidity();
                isValid = false;

                this.dispatchEvent(
                    new ShowToastEvent({
                        title: 'Error',
                        message: 'Please select required fields',
                        variant: 'error'
                    })
                );
            }
        });
        return isValid;
    }

    isDateValid(inputFields){
        let isValid = true;
        inputFields.forEach(inputField => {
            if (inputField.value < this.monthlyStartDate || inputField.value > this.monthlyEndDate) {
                inputField.setCustomValidity("Date should be between Campaign Start Date & Campaign End date");
                isValid = false;
            }else{
                inputField.setCustomValidity("");
            }
            inputField.reportValidity();
        });
        return isValid;
    }

    @api
    handleSubmit(event) {
        console.log('==========================================================================handleSubmit');
        this.showSpinner = true;
        let data = {};
        let blogging = {};
        let emailMarketing = {};
        let socialPosting = {};
        let reviewManagement = {};

        let allMultiSelectValues = {};
        let dateTypeIps = this.template.querySelectorAll('[data-cdate="dateType"]');

        if(this.closedWonOpp && !this.adminScoping){
            this.dispatchEvent(
                new ShowToastEvent({
                    title: 'Error!',
                    message: 'Cannot save scope, Opportunity is already Closed',
                    variant: 'error'
                })
            );
            this.showSpinner = false;
        }else{

        if (this.isDateValid(dateTypeIps) && this.isInputValid() && this.isComboValid() && this.isRadioValid()) {
            try {
                let allMultiSelects = this.template.querySelectorAll('c-multi-select-combobox');
                let allMultiGrid = this.template.querySelectorAll('c-dynamic-multi-input-grid');
                let allMultiGridComp = this.template.querySelectorAll('c-multi-input-grid-comp');
                let contMarketingIps = this.template.querySelectorAll('[data-id="contentMarketPg"]');

                
                for (const cMIp of contMarketingIps) {
                    /*
                    if(cMIp.name=='contentSessionCount'){
                        data[cMIp.name] = cMIp.value;
                        if (cMIp.value>0) {
                            data['sessionData'] = this.sessionData;
                        }
                    }*/
                    if(cMIp.name=='trainingCount'){
                        data[cMIp.name] = cMIp.value;
                        if (cMIp.value>0) {
                            data['trainingSessionData'] = this.trainingSessionData;
                        }
                    }
                    if(cMIp.name=='trainingSession'){
                        data[cMIp.name] = cMIp.value;
                    }
                }
                
                for (const ml of allMultiSelects) {
                    allMultiSelectValues[ml.name] = ml.values;
                    console.log('ml.name'+ml.name);
                }

                if(allMultiSelectValues['trainingType']){
                    data['trainingType']=allMultiSelectValues['trainingType'];
                }
                
                if (allMultiSelectValues['service'].includes('Blogging')) {
                    let allinputs = this.template.querySelectorAll('[data-id="blogging"]');

                    let bloggingSD = this.template.querySelectorAll('[data-id="bloggingSD"]');

                    for (const bsd of bloggingSD) {
                        console.log('inside blogging');
                        if(bsd.name=='contentSessionCount'){
                            blogging[bsd.name] = bsd.value;
                            if (bsd.value>0) {
                                data['bloggingSessionData'] = this.bloggingSessionData;
                            }
                        }
                    }


                    for (const ip of allinputs) {
                        if (ip.value) {
                            blogging[ip.name] = ip.value;
                        }
                    }
                    if(allMultiSelectValues['bloggingTypes']!=null && allMultiSelectValues['bloggingTypes'].length>0){
                        blogging['bloggingTypes']=allMultiSelectValues['bloggingTypes'];
                    }else if(allMultiSelectValues['bloggingTypes']==null || allMultiSelectValues['bloggingTypes'].length==0){
                        this.dispatchEvent(
                            new ShowToastEvent({
                                title: 'Error',
                                message: 'Please select types posted',
                                variant: 'error'
                            })
                        );
                        this.showSpinner=false;
                        return false;
                    }

                    if(allMultiSelectValues['socialPlatforms']!=null){
                        blogging['socialPlatforms']=allMultiSelectValues['socialPlatforms'];
                    }
                    for (const mi of allMultiGrid) {
                        if(mi.name=='bloggingMonthlyData'){
                            blogging[mi.name] = mi.monthlyData();
                        }
                    }

                    for (const mi of allMultiGridComp) {
                        if(mi.name=='miniMonthlyData'){
                            blogging[mi.name] = mi.monthlyData();
                        }
                        if(mi.name=='regularMonthlyData'){
                            blogging[mi.name] = mi.monthlyData();
                        }
                        if(mi.name=='longMonthlyData'){
                            blogging[mi.name] = mi.monthlyData();
                        }
                        if(mi.name=='editedMonthlyData'){
                            blogging[mi.name] = mi.monthlyData();
                        }
                    }

                    data['bloggingRowNames']=this.bloggingRowNames;
                    data['Blogging'] = blogging;
                }
                if (allMultiSelectValues['service'].includes('Email_Marketing')) {
                    let allinputs = this.template.querySelectorAll('[data-id="emailMarketing"]');
                    
                    let emailMarketingSD = this.template.querySelectorAll('[data-id="emailMarketingSD"]');

                    for (const emsd of emailMarketingSD) {
                        if(emsd.name=='contentSessionCount'){
                            emailMarketing[emsd.name] = emsd.value;
                            if (emsd.value>0) {
                                data['emailMarketingSessionData'] = this.emailMarketingSessionData;
                            }
                        }
                    }
                    
                    for (const ip of allinputs) {
                        if (ip.value) {
                            emailMarketing[ip.name] = ip.value;
                        }
                    }
                    if(allMultiSelectValues['campaignTypes']!=null && allMultiSelectValues['campaignTypes'].length>0){
                        emailMarketing['campaignTypes']=allMultiSelectValues['campaignTypes'];
                    }else if(allMultiSelectValues['campaignTypes']==null || allMultiSelectValues['campaignTypes'].length==0){
                        this.dispatchEvent(
                            new ShowToastEvent({
                                title: 'Error',
                                message: 'Please select campaign types',
                                variant: 'error'
                            })
                        );
                        this.showSpinner=false;
                        return false;
                    }

                    for (const mi of allMultiGrid) {
                        if(mi.name=='dripTextMonthlyData'){
                            emailMarketing[mi.name] = mi.monthlyData();
                        }
                        if(mi.name=='dripTextOnlyMonthlyData'){
                            emailMarketing[mi.name] = mi.monthlyData();
                        }
                        if(mi.name=='newsLetterMonthlyData'){
                            emailMarketing[mi.name] = mi.monthlyData();
                        }
                        if(mi.name=='reEngageMonthlyData'){
                            emailMarketing[mi.name] = mi.monthlyData();
                        }
                        if(mi.name=='dripTextOptMonthlyData'){
                            console.log('dripTextOptMonthlyData');
                            emailMarketing[mi.name] = mi.monthlyData();
                        }
                        if(mi.name=='dripTextOnlyOptMonthlyData'){
                            console.log('dripTextOnlyOptMonthlyData');
                            emailMarketing[mi.name] = mi.monthlyData();
                        }
                    }

                    for (const mi of allMultiGridComp) {
                        if(mi.name=='mailChimpMonthlyData'){
                            emailMarketing[mi.name] = mi.monthlyData();
                        }
                    }
                    
                    data['dripTextRowNames']=this.dripTextRowNames;
                    data['dripTextOnlyRowNames']=this.dripTextOnlyRowNames;
                    data['newletterRowNames']=this.newletterRowNames;
                    data['reengagementRowNames']=this.reengagementRowNames;
                    data['EmailMarketing'] = emailMarketing;
                }
                if (allMultiSelectValues['service'].includes('Social_Posting')) {
                    let allinputs = this.template.querySelectorAll('[data-id="socialPosting"]');
                    let socialPostingSD = this.template.querySelectorAll('[data-id="socialPostingSD"]');

                    for (const spsd of socialPostingSD) {
                        if(spsd.name=='contentSessionCount'){
                            socialPosting[spsd.name] = spsd.value;
                            if (spsd.value>0) {
                                data['socPostingSessionData'] = this.socPostingSessionData;
                            }
                        }
                    }


                    for (const ip of allinputs) {
                        if (ip.value) {
                            socialPosting[ip.name] = ip.value;
                        }
                    }
                    if(allMultiSelectValues['socialProfileTypes']!=null && allMultiSelectValues['socialProfileTypes'].length>0){
                        socialPosting['socialProfileTypes']=allMultiSelectValues['socialProfileTypes'];
                    }

                    if(allMultiSelectValues['socialPostingThemes']!=null){
                        socialPosting['socialPostingThemes']=allMultiSelectValues['socialPostingThemes'];
                    }
                    if(allMultiSelectValues['socialPostingPlatform'] && allMultiSelectValues['socialPostingPlatform'].length>0){
                        socialPosting['socialPostingPlatform']=allMultiSelectValues['socialPostingPlatform'];
                    }else if(allMultiSelectValues['socialPostingPlatform'] && allMultiSelectValues['socialPostingPlatform'].length==0){
                        this.dispatchEvent(
                            new ShowToastEvent({
                                title: 'Error',
                                message: 'Please select campaign types',
                                variant: 'error'
                            })
                        );
                        this.showSpinner=false;
                        return false;
                    }

                    for (const mi of allMultiGridComp) {
                        if(mi.name=='communityMgtMonthlyData'){
                            socialPosting[mi.name] = mi.monthlyData();
                        }
                        if(mi.name=='postsMonthlyData'){
                            socialPosting[mi.name] = mi.monthlyData();
                        }
                    }
                    
                    data['SocialPosting'] = socialPosting;
                }
                debugger;
                if (allMultiSelectValues['service'].includes('Review_Management')) {
                    console.log('=======================================================================');
                    for (const mi of allMultiGrid) {
                        if(mi.name=='hoursPerMonthForManagement'){
                            console.log('hoursPerMonthForManagement');
                            reviewManagement[mi.name] = mi.monthlyData();
                        }
                    }
                    data['ReviewManagement'] = reviewManagement;
                }
                console.log(JSON.stringify(data));
                
            } catch (err) {
                console.log(err.message);
                this.showSpinner = false;
            }
            
            if (this.contentMarketing) {
                let scopeNameIp = this.template.querySelector('[data-id="scopeName"]');

                const fields = {};
                fields[ID_FIELD.fieldApiName] = this.recordId;
                fields[JSON_DATA.fieldApiName] = JSON.stringify(data);
                fields[SCOPE_NAME.fieldApiName] = scopeNameIp.value;
                const recordInput = {
                    fields
                };
                updateRecord(recordInput)
                    .then(() => {
                        this.dispatchEvent(
                            new ShowToastEvent({
                                title: 'Success',
                                message: 'Record updated successfully',
                                variant: 'success'
                            })
                        );
                        
                        createActions({
                            recordId: this.recordId
                        }).then(result => {
                            console.log('sucess');
                            this.dispatchEvent(new CustomEvent('createaction', {
                                detail: {
                                    status: 'Success'
                                }
                            }));
                            this.showSpinner = false;
                        }).catch(error => {
                            console.log(error);
                            this.showSpinner = false;
                            let errorMessage = reduceErrors(error);
                            console.log('errorMessage' + errorMessage);
                            this.dispatchEvent(
                                new ShowToastEvent({
                                    title: 'Error creating actions',
                                    message: errorMessage[0],
                                    variant: 'error',
                                    mode: 'sticky'
                                })
                            );
                        })
                    })
                    .catch(error => {
                        let errorMessage = reduceErrors(error);
                        this.dispatchEvent(
                            new ShowToastEvent({
                                title: 'Error updating scope record',
                                message: errorMessage[0],
                                variant: 'error',
                                mode: 'sticky'
                            })
                        );
                        this.showSpinner = false;
                    });
            } 
            
        } else {
            this.showSpinner = false;
        }
    }
    }


}