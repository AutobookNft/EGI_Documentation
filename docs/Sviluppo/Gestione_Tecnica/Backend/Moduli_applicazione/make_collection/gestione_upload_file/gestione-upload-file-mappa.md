# Gestione upload file

## Menu filament: 
pannello: admin
item: collection

### La tabella che mostra le thumbnail degli EGI

url: admin/collection
pages: app/Filament/Pages/Collection.php
view: filament.app.pages.home (@livewire('images'))
component livewire: app/Livewire/Images.php

// Questa è la colonna degli indirizzi dei wallet
WalletCreatorAddress::make('address'),

// Questa è la colonna che contiene i thumbnail delle immagini degli EGI
CollectionImage::make('title')


- WalletCreatorAddress
- CollectionImage

	- componente filament:app/Tables/Columns/CollectionImage.php
view: 'tables.columns.collection-image';
path: (resources/views/tables/columns/collection-image.blade.php)

## Argomento principale 3

## Argomento principale 4

