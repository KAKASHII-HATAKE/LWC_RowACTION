# LWC_RowACTION

https://www.infallibletechie.com/2020/03/lightning-datatable-with-buttonsrow.html


#Component HTML:

<template>      
    <lightning-card title = "Search Accounts" icon-name = "custom:custom63">  
        <div class = "slds-m-around_medium">  
            <lightning-input type = "search" onchange = {handleKeyChange} class = "slds-m-bottom_small" label = "Search" >
            </lightning-input>  
            <template if:true = {accounts}>
                 
                <div style="height: 300px;">  
                    <lightning-datatable key-field="Id"
                                         data={accounts}
                                         columns={columns}
                                         hide-checkbox-column="true"
                                         show-row-number-column="true"
                                         onrowaction={handleRowAction}>
                    </lightning-datatable>  
                </div>   
            </template>      
            <template if:true = {error}>  
                {error}>                  
            </template>  
        </div>  
    </lightning-card>
</template>

#Component JS:

import { LightningElement, track } from 'lwc'; 
import fetchAccounts from '@salesforce/apex/AccountController.fetchAccounts';
import { NavigationMixin } from 'lightning/navigation';

const actions = [
    { label: 'View', name: 'view' },
    { label: 'Edit', name: 'edit' },
];
 
const columns = [   
    { label: 'Name', fieldName: 'Name' }, 
    { label: 'Industry', fieldName: 'Industry' },
    {
        type: 'action',
        typeAttributes: { rowActions: actions },
    }, 
];

export default class LightningDataTableLWC extends NavigationMixin( LightningElement ) {
     
    @track accounts; 
    @track error; 
    @track columns = columns; 
 
    handleKeyChange( event ) { 
         
        const searchKey = event.target.value; 
 
        if ( searchKey ) { 
 
            fetchAccounts( { searchKey } )   
            .then(result => { 
 
                this.accounts = result; 
 
            }) 
            .catch(error => { 
 
                this.error = error; 
 
            }); 
 
        } else 
        this.accounts = undefined; 
 
    }

    handleRowAction( event ) {

        const actionName = event.detail.action.name;
        const row = event.detail.row;
        switch ( actionName ) {
            case 'view':
                this[NavigationMixin.Navigate]({
                    type: 'standard__recordPage',
                    attributes: {
                        recordId: row.Id,
                        actionName: 'view'
                    }
                });
                break;
            case 'edit':
                this[NavigationMixin.Navigate]({
                    type: 'standard__recordPage',
                    attributes: {
                        recordId: row.Id,
                        objectApiName: 'Account',
                        actionName: 'edit'
                    }
                });
                break;
            default:
        }

    }

}

#Apex Class:

public with sharing class AccountController { 
 
    @AuraEnabled( cacheable = true ) 
    public static List< Account > fetchAccounts( String searchKey ) { 
     
        String strKey = '%' + searchKey + '%'; 
        return [ SELECT Id, Name, Industry FROM Account WHERE Name LIKE: strKey LIMIT 10 ]; 
         
    } 
     
}
