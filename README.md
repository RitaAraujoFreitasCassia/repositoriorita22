# repositoriorita22
login.html

<ion-header>
  <ion-toolbar>
    <ion-content padding class="primary-font">
      <form padding [formGroup] = "IoginForm"(submit)="Iogin()" novalidate>
        <ion-item>
          <ion-input [(ngModel)]="email"
          formControlName ="email"
          Type="text"
          Placeholder="E-mail"
          clearInput clearOnEdit="false">
          </ion-input>
          </ion-item>
          <h6 *ngIf="erroEmail" class="error">{{messageEmail}}</h6>
          <ion-item>
            <h6 *ngIf="errorPassword" class="error">{{mensagePassword}}</h6>
            <button ion-button block outline color="secondary" class=""
  </ion-toolbar>
  <ion-navbar>
    <ion-title>login</ion-title>
  </ion-navbar>
</ion-header>

<ion-content padding>

</ion-content>




product
falta fazer


app.module
import { BrowserModule } from '@angular/platform-browser';
import { ErrorHandler, NgModule } from '@angular/core';
import { IonicApp, IonicErrorHandler, IonicModule } from 'ionic-angular';
import { SplashScreen } from '@ionic-native/splash-screen';
import { StatusBar } from '@ionic-native/status-bar';
 
import { MyApp } from './app.component';
import { HomePage } from '../pages/home/home';
 
import { IonicStorageModule } from '@ionic/storage';
import { DatePipe } from '@angular/common';
import { ContactProvider, Contact } from '../pages/contact/contact';

 
@NgModule({
  declarations: [
    MyApp,
    HomePage,
    ContactProvider,
    Contact
  ],
  imports: [
    BrowserModule,
    IonicModule.forRoot(MyApp),
    IonicStorageModule.forRoot()
  ],
  bootstrap: [IonicApp],
  entryComponents: [
    MyApp,
    HomePage
  ],
  providers: [
    StatusBar,
    SplashScreen,
    {provide: ErrorHandler, useClass: IonicErrorHandler},
    DatePipe,
    ContactProvider
  ]
})
export class AppModule {
}


edit-contact
import { Component } from '@angular/core';
import { IonicPage, NavController, NavParams, ToastController } from 'ionic-angular';
import { ContactProvider, Contact } from '../../providers/contact/contact';

@IonicPage()
@Component({
  selector: 'page-edit-contact',
  templateUrl: 'edit-contact.html',
})
export class EditContactPage {
  model: Contact;
  key: string;

  constructor(public navCtrl: NavController, public navParams: NavParams, private contactProvider: ContactProvider, private toast: ToastController) {
    if (this.navParams.data.contact && this.navParams.data.key) {
      this.model = this.navParams.data.contact;
      this.key =  this.navParams.data.key;
    } else {
      this.model = new Contact();
    }
  }

  save() {
    this.saveContact()
      .then(() => {
        this.toast.create({ message: 'Contato salvo.', duration: 3000, position: 'botton' }).present();
        this.navCtrl.pop();
      })
      .catch(() => {
        this.toast.create({ message: 'Erro ao salvar o contato.', duration: 3000, position: 'botton' }).present();
      });
  }

  private saveContact() {
    if (this.key) {
      return this.contactProvider.update(this.key, this.model);
    } else {
      return this.contactProvider.insert(this.model);
    }
  }

}


editr-contact.html
<ion-header>
    <ion-navbar>
        <ion-title>
          Ionic Storage Example
        </ion-title>
      </ion-navbar>
    </ion-header>
     
    <ion-content padding>
     
      <ion-list>
     
        <ion-item>
          <ion-label stacked>Nome</ion-label>
          <ion-input type="text" name="name" [(ngModel)]="model.name"></ion-input>
        </ion-item>
     
        <ion-item>
          <ion-label stacked>Telefone</ion-label>
          <ion-input type="tel" name="phone" [(ngModel)]="model.phone"></ion-input>
        </ion-item>
     
        <ion-item>
          <ion-label stacked>Nascimento</ion-label>
          <ion-datetime displayFormat="DD/MM/YYYY" name="birth" [(ngModel)]="model.birth"></ion-datetime>
        </ion-item>
     
        <ion-item>
          <ion-label>Ativo</ion-label>
          <ion-checkbox name="active" [(ngModel)]="model.active"></ion-checkbox>
        </ion-item>
     
      </ion-list>
     
      <button ion-button block (click)="save()">Salvar</button>
     
    </ion-content>
    
    home.ts
    import { Component } from '@angular/core';
import { NavController, ToastController } from 'ionic-angular';
import { ContactProvider, Contact, ContactList } from '../../providers/contact/contact';

@Component({
  selector: 'page-home',
  templateUrl: 'home.html'
})
export class HomePage {
  contacts: ContactList[];

  constructor(public navCtrl: NavController, private contactProvider: ContactProvider, private toast: ToastController) { }

  ionViewDidEnter() {
    this.contactProvider.getAll()
      .then((result) => {
        this.contacts = result;
      });
  }

  addContact() {
    this.navCtrl.push('EditContactPage');
  }

  editContact(item: ContactList) {
    this.navCtrl.push('EditContactPage', { key: item.key, contact: item.contact });
  }

  removeContact(item: ContactList) {
    this.contactProvider.remove(item.key)
      .then(() => {
        // Removendo do array de items
        var index = this.contacts.indexOf(item);
        this.contacts.splice(index, 1);
        this.toast.create({ message: 'Contato removido.', duration: 3000, position: 'botton' }).present();
      })
  }

}

contact.ts
import { Injectable } from '@angular/core';
import 'rxjs/add/operator/map';
import { Storage } from '@ionic/storage';
import { DatePipe } from '@angular/common';
 
@Injectable()
export class ContactProvider {
 
  constructor(private storage: Storage, private datepipe: DatePipe) { }
 
  public insert(contact: Contact) {
    let key = this.datepipe.transform(new Date(), "ddMMyyyyHHmmss");
    return this.save(key, contact);
  }
 
  public update(key: string, contact: Contact) {
    return this.save(key, contact);
  }
 
  private save(key: string, contact: Contact) {
    return this.storage.set(key, contact);
  }
 
  public remove(key: string) {
    return this.storage.remove(key);
  }
 
  public getAll() {
 
    let contacts: ContactList[] = [];
 
    return this.storage.forEach((value: Contact, key: string, iterationNumber: Number) => {
      let contact = new ContactList();
      contact.key = key;
      contact.contact = value;
      contacts.push(contact);
    })
      .then(() => {
        return Promise.resolve(contacts);
      })
      .catch((error) => {
        return Promise.reject(error);
      });
  }
}
 
export class Contact {
  name: string;
  phone: number;
  birth: Date;
  active: boolean;
}
 
export class ContactList {
  key: string;
  contact: Contact;
}
